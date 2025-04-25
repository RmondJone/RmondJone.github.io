---
title: "Android ANR日志分析"
date: 2025-04-25T17:14:01+08:00
draft: false
categories: ["移动端"]
tags: ["Android"]
---

# Android ANR 日志分析方法指南

ANR (Application Not Responding) 是 Android 应用中常见的问题，当主线程被阻塞超过一定时间（通常5秒）时触发。以下是分析 ANR 日志的系统方法：

## 一、获取 ANR 日志

### 1. 通过 adb 获取
```bash
adb pull /data/anr/traces.txt
```

### 2. 从设备直接获取
路径一般为：
- `/data/anr/traces.txt` (Android 10及以下)
- `/data/anr/anr_*` (Android 11及以上)

## 二、ANR 日志关键部分解析

一个典型的 ANR 日志包含以下重要信息：

```
----- pid 12345 at 2023-01-01 12:00:00 -----
Cmd line: com.example.app  # 发生ANR的包名
...
DALVIK THREADS (12):
"main" prio=5 tid=1 Native
  | group="main" sCount=1 dsCount=0 flags=1 obj=0x72e8f7d0 self=0x7f88d4a400
  | sysTid=12345 nice=0 cgrp=default sched=0/0 handle=0x7fa0a8a4f0
  | state=S schedstat=( 123456789 987654321 123 ) utm=12 stm=34 core=1 HZ=100
  | stack=0x7fc3a3a000-0x7fc3a3c000 stackSize=8192KB
  | held mutexes=
  at java.lang.Object.wait(Native method)
  - waiting on <0x0cd5a1d2> (a java.lang.Object)
  at java.lang.Thread.parkFor$(Thread.java:2137)
  - locked <0x0cd5a1d2> (a java.lang.Object)
  at sun.misc.Unsafe.park(Unsafe.java:358)
  at java.util.concurrent.locks.LockSupport.park(LockSupport.java:190)
  at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:868)
  at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireSharedInterruptibly(AbstractQueuedSynchronizer.java:1021)
  at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireSharedInterruptibly(AbstractQueuedSynchronizer.java:1328)
  at java.util.concurrent.CountDownLatch.await(CountDownLatch.java:232)
  at com.example.app.MainActivity.doHeavyTask(MainActivity.java:123)
  ...
```

## 三、分析步骤

### 1. 确认 ANR 类型
查看日志开头的 ANR 原因：
- `Input dispatching timed out`：输入事件超时
- `Broadcast of Intent`：广播处理超时
- `executing service`：服务执行超时
- `ContentProvider not responding`：内容提供者响应超时

### 2. 定位主线程堆栈
查找标记为 `"main"` 的线程，这是分析的重点：

```
"main" prio=5 tid=1 Native
  | group="main" sCount=1 dsCount=0 flags=1 obj=0x72e8f7d0 self=0x7f88d4a400
  | state=S schedstat=( ... ) utm=12 stm=34 core=1
  | stack=... stackSize=8192KB
  | held mutexes=
  at java.lang.Object.wait(Native method)
  - waiting on <0x0cd5a1d2> (a java.lang.Object)
  at com.example.app.MainActivity.doHeavyTask(MainActivity.java:123)
```

### 3. 分析阻塞原因

常见阻塞模式及解决方案：

| 阻塞现象 | 可能原因 | 解决方案 |
|---------|---------|---------|
| 主线程在等待锁 | 死锁或同步锁竞争 | 检查同步代码，改用并发数据结构 |
| 主线程执行IO操作 | 文件/网络操作 | 移到子线程，使用AsyncTask/RxJava/协程 |
| 主线程执行数据库操作 | 复杂查询或大量数据 | 使用Room的异步查询，优化数据库 |
| 主线程执行计算 | 复杂算法/大量数据处理 | 使用子线程计算 |
| Binder调用阻塞 | 跨进程通信耗时 | 优化服务端实现，减少数据传输量 |

### 4. 检查系统负载

查看日志中的系统状态：
```
CPU usage from 0ms to 10000ms later:
  50% 1234/system_server: 30% user + 20% kernel / faults: 100 minor
  30% 5678/com.example.app: 20% user + 10% kernel / faults: 50 minor
  20% 9101/com.android.phone: 15% user + 5% kernel
```

高CPU使用率可能表明：
- 系统资源不足
- 应用存在CPU密集型任务
- 其他应用占用了过多资源

## 四、高级分析工具

### 1. Android Studio Profiler
- CPU Profiler 分析主线程执行
- Memory Profiler 检查内存问题

### 2. StrictMode
```java
StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
    .detectDiskReads()
    .detectDiskWrites()
    .detectNetwork()
    .penaltyLog()
    .build());
```

### 3. 使用 Bugreport
```bash
adb bugreport
```
分析更完整的系统状态信息

## 五、预防 ANR 的最佳实践

1. **主线程只做UI操作**：所有耗时操作移到子线程
2. **优化BroadcastReceiver**：在onReceive()中避免耗时操作
3. **使用适当线程模型**：
    - 轻量级任务：HandlerThread
    - 复杂异步：RxJava/协程
    - 数据库：Room + LiveData
4. **避免死锁**：谨慎使用同步锁
5. **监控ANR**：实现UncaughtExceptionHandler捕获潜在ANR

## 六、常见问题排查示例

**案例1：数据库操作阻塞**
```
"main" prio=5 tid=1 Native
  at android.database.sqlite.SQLiteConnection.nativeExecuteForCursorWindow
  at com.example.app.UserDao.loadAllUsers(UserDao.java:45)
```
→ 解决方案：使用Room的异步查询或CursorLoader

**案例2：网络请求阻塞**
```
"main" prio=5 tid=1 Runnable
  at okhttp3.internal.http.RetryAndFollowUpInterceptor.intercept
  at com.example.app.NetworkHelper.fetchData(NetworkHelper.java:33)
```
→ 解决方案：使用Retrofit + RxJava/Coroutines

通过系统分析ANR日志，可以准确定位问题根源并采取针对性优化措施。
