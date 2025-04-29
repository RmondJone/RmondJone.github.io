---
title: "Kotlin 协程：使用与原理详解"
date: 2025-04-29T13:26:09+08:00
draft: false
categories: ["移动端"]
tags: ["Kotlin","Android"]
---

## 一、Kotlin 协程基本概念

### 1. 什么是协程

协程（Coroutine）是 Kotlin 提供的一种轻量级线程管理框架，它允许以顺序的方式编写异步代码，同时提供了高效的并发处理能力。

**核心特点**：
- **轻量级**：比线程更高效，可以创建数千个协程而不会导致性能问题
- **结构化并发**：提供明确的生命周期管理和取消机制
- **挂起机制**：可以在不阻塞线程的情况下暂停和恢复执行

### 2. 协程 vs 线程

| 特性          | 协程                          | 线程                          |
|--------------|------------------------------|------------------------------|
| 创建成本       | 非常低（字节级）               | 较高（MB级栈内存）             |
| 切换开销       | 几乎为零                      | 需要内核介入，开销较大          |
| 并发数量       | 可轻松创建数万个               | 通常数百个就达到极限            |
| 阻塞行为       | 挂起不阻塞线程                 | 阻塞会占用线程资源              |

## 二、协程的基本使用

### 1. 添加依赖

```kotlin
// build.gradle (Module)
dependencies {
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.6.4'
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.6.4'
}
```

### 2. 启动协程的三种方式

#### (1) `launch` - 启动不需要返回结果的协程

```kotlin
// 在Activity或ViewModel中
lifecycleScope.launch {
    // 在主线程执行
    val result = fetchData() // 挂起函数
    updateUI(result)
}

private suspend fun fetchData(): String {
    return withContext(Dispatchers.IO) {
        // 在IO线程执行
        delay(1000) // 模拟耗时操作
        "Data loaded"
    }
}
```

#### (2) `async` - 启动需要返回结果的协程

```kotlin
lifecycleScope.launch {
    val deferred1 = async { fetchData1() }
    val deferred2 = async { fetchData2() }
    
    // await()是挂起函数，不会阻塞线程
    val result1 = deferred1.await()
    val result2 = deferred2.await()
    
    showResults(result1, result2)
}
```

#### (3) `runBlocking` - 阻塞当前线程的协程（主要用于测试）

```kotlin
fun main() = runBlocking {
    // 会阻塞当前线程直到协程完成
    launch {
        delay(1000L)
        println("World!")
    }
    println("Hello,")
}
```

### 3. 协程调度器（Dispatchers）

| 调度器                | 用途                          |
|----------------------|------------------------------|
| `Dispatchers.Main`   | Android主线程更新UI           |
| `Dispatchers.IO`     | 磁盘/网络IO操作                |
| `Dispatchers.Default`| CPU密集型计算任务              |
| `Dispatchers.Unconfined` | 不限定特定线程（不推荐常规使用） |

```kotlin
viewModelScope.launch(Dispatchers.IO) {
    // 在IO线程执行
    val data = repository.loadData()
    
    withContext(Dispatchers.Main) {
        // 切换回主线程更新UI
        _uiState.value = UiState.Success(data)
    }
}
```

## 三、协程的核心原理

### 1. 挂起函数（Suspend Function）原理

挂起函数是协程的核心机制，编译器会对挂起函数进行特殊处理：

```kotlin
suspend fun fetchUser(): User {
    val user = api.fetchUser() // 另一个挂起函数
    return user
}
```

**编译后的伪代码**：

```kotlin
Object fetchUser(Continuation<User> continuation) {
    // 状态机实现
    when(continuation.label) {
        0 -> {
            continuation.label = 1
            api.fetchUser(continuation)
            return COROUTINE_SUSPENDED
        }
        1 -> {
            val user = continuation.result as User
            return user
        }
    }
}
```

### 2. 协程的 CPS (Continuation Passing Style) 变换

Kotlin 编译器会将挂起函数转换为**状态机**，通过 `Continuation` 回调实现挂起和恢复：

1. 每次挂起点（suspend point）都会成为状态机的一个状态
2. 挂起时保存当前状态和局部变量
3. 恢复时从保存的状态继续执行

### 3. 协程上下文（CoroutineContext）

协程上下文包含以下关键元素：

- **Job**：控制协程的生命周期和取消
- **CoroutineDispatcher**：决定协程在哪个线程执行
- **CoroutineName**：协程的名称（调试用）
- **CoroutineExceptionHandler**：异常处理

```kotlin
val customContext = Dispatchers.IO + CoroutineName("MyCoroutine") + exceptionHandler

lifecycleScope.launch(customContext) {
    // 使用自定义上下文
}
```

### 4. 结构化并发

Kotlin 协程通过以下机制实现结构化并发：

- **父协程-子协程关系**：父协程会等待所有子协程完成
- **取消传播**：取消父协程会自动取消所有子协程
- **异常传播**：子协程异常会传播给父协程

```kotlin
val parentJob = CoroutineScope(Dispatchers.Main).launch {
    // 父协程
    launch {
        // 子协程1
        delay(1000)
    }
    launch {
        // 子协程2
        delay(2000)
    }
}

// 取消父协程会同时取消两个子协程
parentJob.cancel()
```

## 四、Android 中的协程最佳实践

### 1. 使用 Jetpack 提供的协程作用域

```kotlin
// Activity/Fragment
lifecycleScope.launch {
    // 自动跟随生命周期取消
}

// ViewModel
viewModelScope.launch {
    // 当ViewModel cleared时自动取消
}
```

### 2. 正确处理协程异常

```kotlin
// 方式1：try-catch
lifecycleScope.launch {
    try {
        fetchData()
    } catch (e: Exception) {
        showError(e)
    }
}

// 方式2：CoroutineExceptionHandler
val handler = CoroutineExceptionHandler { _, exception ->
    Log.e("Coroutine", "Caught $exception")
}

lifecycleScope.launch(handler) {
    throw RuntimeException("Test exception")
}
```

### 3. 避免内存泄漏

```kotlin
// 错误示例：可能泄漏Activity
class MyActivity : AppCompatActivity() {
    private var job: Job? = null
    
    override fun onCreate() {
        job = GlobalScope.launch {
            // 即使Activity销毁，协程仍继续运行
            fetchData()
        }
    }
}

// 正确示例：使用lifecycleScope
class MyActivity : AppCompatActivity() {
    override fun onCreate() {
        lifecycleScope.launch {
            // 当Activity销毁时自动取消
            fetchData()
        }
    }
}
```

### 4. 协程与 Retrofit 结合

```kotlin
interface ApiService {
    @GET("users/{id}")
    suspend fun getUser(@Path("id") id: String): User // 挂起函数
}

// ViewModel中
fun loadUser(userId: String) {
    viewModelScope.launch {
        try {
            _uiState.value = UiState.Loading
            val user = repository.getUser(userId)
            _uiState.value = UiState.Success(user)
        } catch (e: Exception) {
            _uiState.value = UiState.Error(e)
        }
    }
}
```

## 五、协程高级主题

### 1. 通道（Channel）

```kotlin
val channel = Channel<Int>()

// 生产者
lifecycleScope.launch {
    for (i in 1..5) {
        delay(100)
        channel.send(i)
    }
    channel.close()
}

// 消费者
lifecycleScope.launch {
    for (value in channel) {
        println("Received $value")
    }
}
```

### 2. 流（Flow）

```kotlin
fun fetchDataFlow(): Flow<String> = flow {
    for (i in 1..5) {
        delay(1000)
        emit("Data $i")
    }
}

// 收集流
lifecycleScope.launch {
    fetchDataFlow()
        .flowOn(Dispatchers.IO) // 指定上游执行上下文
        .catch { e -> println("Error: $e") }
        .collect { value ->
            // 在主线程接收数据
            updateUI(value)
        }
}
```

### 3. 协程测试

```kotlin
@Test
fun testCoroutine() = runTest { // 使用TestCoroutineScope
    val repository = TestRepository()
    
    // 启动协程
    val job = launch {
        repository.loadData()
    }
    
    // 控制虚拟时间
    advanceTimeBy(1000)
    
    // 验证结果
    assertEquals("Test Data", repository.result)
    
    // 确保协程完成
    job.cancel()
}
```

## 六、协程性能优化

1. **避免过度切换调度器**：减少 `withContext` 切换
2. **合理使用协程作用域**：避免不必要的协程存活
3. **适当使用 `flowOn`**：减少 Flow 的线程切换
4. **使用 `supervisorScope`**：当不希望子协程异常影响其他子协程时
5. **合理设置协程上下文**：避免不必要的上下文元素

## 总结

Kotlin 协程为 Android 开发提供了强大的异步编程解决方案：

- **简化异步代码**：用同步方式写异步逻辑
- **高效线程管理**：减少线程切换开销
- **结构化并发**：避免内存泄漏和资源浪费
- **与Jetpack深度集成**：`lifecycleScope`、`viewModelScope`等

理解协程的原理和最佳实践，可以帮助开发者编写更高效、更健壮的 Android 应用。