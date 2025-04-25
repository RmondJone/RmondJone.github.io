---
title: "Android Service 详解"
date: 2025-04-25T15:51:38+08:00
draft: false
categories: ["移动端"]
tags: ["Android"]
---

# Android Service 详解

Service 是 Android 四大组件之一，主要用于在后台执行长时间运行的操作，或者为其他应用提供功能服务。Service 不提供用户界面，但可以在后台持续运行，即使应用切换到后台或用户切换到其他应用。

## Service 的基本特点

1. **后台运行**：没有用户界面，适合执行不需要用户交互的操作
2. **长期存活**：比 Activity 生命周期更长，不受 Activity 生命周期直接影响
3. **主线程运行**：默认运行在主线程，需要手动创建新线程执行耗时操作

## Service 的类型

### 1. 按启动方式分类

#### (1) Started Service（启动服务）

- 通过 `startService()` 启动
- 会一直运行，直到调用 `stopSelf()` 或其他组件调用 `stopService()`
- 不直接与启动它的组件通信
- 适合执行不需要返回结果的操作（如下载文件）

#### (2) Bound Service（绑定服务）

- 通过 `bindService()` 启动
- 提供客户端-服务器接口，允许组件与 Service 交互
- 多个组件可以绑定到同一个 Service
- 当所有绑定的组件都解绑后，Service 会被销毁
- 适合需要交互的场景（如音乐播放器）

### 2. 按运行位置分类

#### (1) 本地服务 (Local Service)

- 运行在应用程序进程的主线程中
- 只能被同一应用内的组件访问

#### (2) 远程服务 (Remote Service)

- 运行在独立的进程中
- 可以通过 AIDL (Android Interface Definition Language) 定义接口供其他应用访问

## Service 生命周期

Service 的生命周期根据启动方式不同而有所差异：

![](images/android_service.png)

### Started Service 生命周期

1. `onCreate()` - Service 首次创建时调用
2. `onStartCommand()` - 每次通过 `startService()` 启动时调用
3. `onDestroy()` - Service 终止时调用

### Bound Service 生命周期

1. `onCreate()` - Service 首次创建时调用
2. `onBind()` - 当组件通过 `bindService()` 绑定时调用
3. `onUnbind()` - 当所有组件都解绑时调用
4. `onDestroy()` - Service 终止时调用

## 实现 Service

### 1. 创建 Service 类

```java
public class MyService extends Service {
    private static final String TAG = "MyService";
    
    // 必须实现的方法
    @Override
    public IBinder onBind(Intent intent) {
        // 绑定服务时调用
        return null; // 对于启动服务返回 null
    }
    
    @Override
    public void onCreate() {
        super.onCreate();
        Log.d(TAG, "onCreate called");
    }
    
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.d(TAG, "onStartCommand called");
        // 执行任务...
        return START_STICKY; // 定义服务被杀死后的行为
    }
    
    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d(TAG, "onDestroy called");
    }
}
```

### 2. 在 AndroidManifest.xml 中声明

```xml
<service android:name=".MyService" />
```

## 启动和停止 Service

### 启动 Service

```java
Intent serviceIntent = new Intent(this, MyService.class);
startService(serviceIntent); // 启动服务
// 或
bindService(serviceIntent, serviceConnection, Context.BIND_AUTO_CREATE); // 绑定服务
```

### 停止 Service

```java
Intent serviceIntent = new Intent(this, MyService.class);
stopService(serviceIntent); // 停止启动的服务
// 或
unbindService(serviceConnection); // 解绑服务
```

## IntentService

IntentService 是 Service 的子类，简化了启动服务的实现：

- 自动创建工作线程执行任务
- 任务执行完成后自动停止服务
- 适合执行一次性后台任务

```java
public class MyIntentService extends IntentService {
    public MyIntentService() {
        super("MyIntentService");
    }
    
    @Override
    protected void onHandleIntent(@Nullable Intent intent) {
        // 在这里执行后台任务
    }
}
```

## 前台服务 (Foreground Service)

Android 8.0 (Oreo) 后对后台服务限制增加，前台服务成为更好的选择：

1. 必须显示通知
2. 优先级更高，不易被系统杀死
3. 适合需要持续运行且用户感知的服务（如音乐播放、GPS跟踪）

```java
// 创建通知
Notification notification = new Notification.Builder(this, CHANNEL_ID)
        .setContentTitle("服务运行中")
        .setContentText("正在执行重要任务")
        .setSmallIcon(R.drawable.ic_notification)
        .build();

// 启动为前台服务
startForeground(NOTIFICATION_ID, notification);
```

## 绑定服务的实现

绑定服务通常用于实现进程间通信(IPC)：

### 1. 创建 Binder

```java
public class LocalBinder extends Binder {
    MyService getService() {
        return MyService.this;
    }
}

private final IBinder binder = new LocalBinder();

@Override
public IBinder onBind(Intent intent) {
    return binder;
}
```

### 2. 实现 ServiceConnection

```java
private ServiceConnection connection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        LocalBinder binder = (LocalBinder) service;
        myService = binder.getService();
        isBound = true;
    }
    
    @Override
    public void onServiceDisconnected(ComponentName name) {
        isBound = false;
    }
};
```

## 服务与线程的选择

| 场景 | 选择 |
|------|------|
| 短期后台任务 | AsyncTask 或 Thread |
| 需要跨Activity存活的任务 | Service |
| 需要系统高优先级运行 | 前台服务 |
| 需要与其他应用共享功能 | 绑定服务+AIDL |

## 注意事项

1. **ANR 问题**：Service 默认运行在主线程，耗时操作必须开子线程
2. **内存泄漏**：绑定服务时要记得解绑
3. **后台限制**：Android 8.0+ 对后台服务有限制，考虑使用 JobScheduler
4. **电量优化**：长时间运行的服务会影响电池寿命
5. **权限**：某些操作需要声明权限（如前台服务需要 `FOREGROUND_SERVICE`）

## 最佳实践

1. 尽量使用 JobScheduler/WorkManager 替代长时间运行的服务
2. 使用前台服务时提供清晰的通知和关闭选项
3. 及时释放资源，在 `onDestroy()` 中清理
4. 考虑使用 `Sticky` 服务保证重要服务被杀死后能重启
5. 对于跨进程通信，考虑使用 Messenger 简化 AIDL 实现

Service 是 Android 强大的后台处理机制，合理使用可以提升应用体验，但滥用会导致性能问题，需要根据实际需求选择最合适的实现方式。
