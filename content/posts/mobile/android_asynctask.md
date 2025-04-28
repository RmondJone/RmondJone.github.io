---
title: "Android AsyncTask详解"
date: 2025-04-28T09:26:56+08:00
draft: false
categories: ["移动端"]
tags: ["Android"]
mermaid: true
---

## **AsyncTask 原理深度解析**

AsyncTask 是 Android 早期提供的一个轻量级异步任务框架，封装了 `Thread` + `Handler`，用于在后台线程执行任务并更新 UI。但由于其设计缺陷（如内存泄漏、平台兼容性问题），Google 已标记为 **`@Deprecated`**（Android 11+ 正式废弃），但理解其原理仍对面试和代码优化有帮助。

### **1. AsyncTask 核心组成**
AsyncTask 主要涉及 **3 个泛型参数** 和 **4 个核心方法**：
```java
public abstract class AsyncTask<Params, Progress, Result> {
    // 泛型参数：
    // - Params: 任务输入参数类型（如 URL）
    // - Progress: 进度更新类型（如 Integer）
    // - Result: 任务结果类型（如 Bitmap）

    // 核心方法：
    protected abstract Result doInBackground(Params... params); // 【子线程】执行耗时任务
    protected void onPreExecute() {}          // 【主线程】任务开始前调用（如显示进度条）
    protected void onProgressUpdate(Progress... values) {} // 【主线程】更新进度
    protected void onPostExecute(Result result) {} // 【主线程】任务完成后调用（如更新UI）
}
```

### **2. AsyncTask 内部工作原理**
#### **（1）线程调度模型**
AsyncTask 内部使用 **2 个线程池 + 1 个 Handler**：
- **`SerialExecutor`（串行任务队列，Android 3.0+ 默认）**  
  保证任务按提交顺序依次执行（类似单线程池）。
- **`THREAD_POOL_EXECUTOR`（并行线程池，可通过 `executeOnExecutor()` 手动指定）**  
  允许任务并行执行（但早期版本存在并发问题）。
- **`InternalHandler`（主线程 Handler）**  
  用于从后台线程切换回主线程（调用 `onProgressUpdate` / `onPostExecute`）。

#### **（2）任务执行流程**

{{<mermaid >}}
sequenceDiagram
    participant 主线程
    participant SerialExecutor
    participant ThreadPoolExecutor
    participant 子线程
    participant InternalHandler

    主线程->>SerialExecutor: execute(AsyncTask)
    SerialExecutor->>ThreadPoolExecutor: 提交任务到线程池
    ThreadPoolExecutor->>子线程: 执行doInBackground()
    子线程->>InternalHandler: 发送进度消息（updateProgress）
    InternalHandler->>主线程: onProgressUpdate()
    子线程->>InternalHandler: 发送结果消息（postResult）
    InternalHandler->>主线程: onPostExecute()
{{</mermaid >}}

### **3. 关键源码分析**
#### **（1）任务提交**
```java
public final AsyncTask<Params, Progress, Result> execute(Params... params) {
    return executeOnExecutor(sDefaultExecutor, params); // 默认使用 SerialExecutor
}

public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec, Params... params) {
    // 状态检查（不能重复执行）
    if (mStatus != Status.PENDING) throw new IllegalStateException("Already executed");
    mStatus = Status.RUNNING;
    onPreExecute(); // 【主线程】预处理
    mWorker.mParams = params; // 保存参数
    exec.execute(mFuture); // 提交到线程池
    return this;
}
```

#### **（2）SerialExecutor（串行执行器）**
```java
private static class SerialExecutor implements Executor {
    final ArrayDeque<Runnable> mTasks = new ArrayDeque<>(); // 双端队列存储任务
    Runnable mActive;

    public synchronized void execute(final Runnable r) {
        mTasks.offer(new Runnable() { // 将任务加入队列
            public void run() {
                try {
                    r.run(); // 实际执行任务
                } finally {
                    scheduleNext(); // 完成后触发下一个任务
                }
            }
        });
        if (mActive == null) scheduleNext(); // 首次执行
    }

    synchronized void scheduleNext() {
        if ((mActive = mTasks.poll()) != null) {
            THREAD_POOL_EXECUTOR.execute(mActive); // 交给线程池执行
        }
    }
}
```

#### **（3）InternalHandler（主线程通信）**
```java
private static class InternalHandler extends Handler {
    public InternalHandler(Looper looper) {
        super(looper);
    }

    @Override
    public void handleMessage(Message msg) {
        AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
        switch (msg.what) {
            case MESSAGE_POST_RESULT:
                result.mTask.finish(result.mData[0]); // 调用 onPostExecute
                break;
            case MESSAGE_POST_PROGRESS:
                result.mTask.onProgressUpdate(result.mData); // 更新进度
                break;
        }
    }
}
```

### **4. AsyncTask 的缺陷**
#### **（1）内存泄漏**
- **问题**：AsyncTask 持有 Activity 的隐式引用（如匿名内部类），若任务未完成而 Activity 销毁，会导致内存泄漏。
- **解决**：使用静态内部类 + WeakReference，或在 `onDestroy()` 中调用 `cancel(true)`。

#### **（2）串行执行效率低**
- **问题**：Android 3.0+ 默认使用 `SerialExecutor`，任务排队执行。
- **解决**：手动指定 `THREAD_POOL_EXECUTOR`：
  ```java
  task.executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR, params);
  ```

#### **（3）平台兼容性问题**
- **问题**：不同 Android 版本线程池行为不一致（如早期版本并行执行）。
- **解决**：直接使用 `ThreadPoolExecutor` 或 `RxJava` / `Kotlin 协程` 替代。

### **5. 替代方案**
| 方案                | 优点                          | 缺点                          |
|---------------------|-----------------------------|-----------------------------|
| **ThreadPoolExecutor** | 灵活控制线程池参数             | 需手动处理线程切换            |
| **RxJava**          | 链式调用、线程切换方便         | 学习成本高                   |
| **Kotlin 协程**     | 代码简洁、结构化并发           | 需适配 Kotlin 项目           |
| **LiveData**        | 生命周期感知、自动更新 UI      | 仅适合观察数据变化，不适用耗时任务 |

### **6. 面试高频问题**
1. **AsyncTask 为什么在 Android 11 被废弃？**  
   答：因设计缺陷（内存泄漏、串行执行效率低、平台兼容性问题），推荐使用 `ExecutorService` + `Handler` 或现代框架（如协程）。

2. **AsyncTask 的默认执行模式是并行还是串行？**  
   答：Android 1.6~2.3 默认并行（`THREAD_POOL_EXECUTOR`），3.0+ 改为串行（`SerialExecutor`）。

3. **如何让 AsyncTask 并行执行？**  
   答：调用 `executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR)`。

4. **onPostExecute 一定在主线程执行吗？为什么？**  
   答：是，因内部通过 `InternalHandler` 绑定主线程 Looper 发送消息。

通过深入理解 AsyncTask 的源码和缺陷，可以更好地应对面试中的并发问题，并在实际开发中选择合适的替代方案。
