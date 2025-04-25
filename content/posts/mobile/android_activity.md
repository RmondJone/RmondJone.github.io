---
title: "Android Activity详解"
date: 2025-04-25T10:08:56+08:00
draft: false
categories: ["移动端"]
tags: ["Android"]
---

# Android Activity 详解

Activity 是 Android 应用的核心组件之一，它代表了一个具有用户界面的单一屏幕。以下是关于 Android Activity 的全面解析：

## 一、Activity 基本概念

1. **定义**：Activity 是 Android 应用的呈现层，每个屏幕都是一个 Activity。
2. **作用**：负责与用户交互，显示界面内容，处理用户输入。
3. **特点**：
    - 一个应用通常由多个 Activity 组成
    - Activity 之间可以相互调用
    - 每个 Activity 都有独立的生命周期

## 二、Activity 生命周期

Activity 生命周期是理解 Activity 的核心，包含以下回调方法：

```java
public class MainActivity extends Activity {
    // 创建时调用
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
    
    // 即将可见时调用
    @Override
    protected void onStart() {
        super.onStart();
    }
    
    // 可与用户交互时调用
    @Override
    protected void onResume() {
        super.onResume();
    }
    
    // 失去焦点但仍可见时调用
    @Override
    protected void onPause() {
        super.onPause();
    }
    
    // 完全不可见时调用
    @Override
    protected void onStop() {
        super.onStop();
    }
    
    // 被销毁前调用
    @Override
    protected void onDestroy() {
        super.onDestroy();
    }
    
    // 从停止状态恢复时调用
    @Override
    protected void onRestart() {
        super.onRestart();
    }
}
```

### 生命周期图示

```
启动 → onCreate → onStart → onResume → 运行状态
运行状态 → onPause → onStop → 停止状态
停止状态 → onRestart → onStart → onResume → 运行状态
运行状态 → onPause → onStop → onDestroy → 销毁状态
```

![](/images/activity_lifecycle.png)

## 三、Activity 的启动与传参

### 1. 显式启动

```java
// 启动另一个Activity
Intent intent = new Intent(this, SecondActivity.class);
startActivity(intent);
```

### 2. 隐式启动

```java
Intent intent = new Intent(Intent.ACTION_VIEW);
intent.setData(Uri.parse("http://www.example.com"));
startActivity(intent);
```

### 3. 传递数据

发送数据：
```java
Intent intent = new Intent(this, SecondActivity.class);
intent.putExtra("key", "value");
startActivity(intent);
```

接收数据：
```java
String value = getIntent().getStringExtra("key");
```

### 4. 返回结果

启动Activity并期待返回结果：
```java
startActivityForResult(intent, REQUEST_CODE);
```

处理返回结果：
```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == REQUEST_CODE && resultCode == RESULT_OK) {
        // 处理返回数据
    }
}
```

设置返回结果：
```java
Intent resultIntent = new Intent();
resultIntent.putExtra("result", "some data");
setResult(RESULT_OK, resultIntent);
finish();
```

## 四、Activity 的启动模式

在 AndroidManifest.xml 中配置：

```xml
<activity android:name=".MainActivity"
    android:launchMode="standard" />
```

四种启动模式：

1. **standard**（默认）：每次启动都创建新实例
2. **singleTop**：如果已在栈顶则不创建新实例
3. **singleTask**：整个任务栈中只保留一个实例
4. **singleInstance**：单独存在于一个任务栈中

## 五、Activity 的任务栈（Task）

- 每个应用有自己的任务栈
- Activity 按照"后进先出"的顺序排列
- 可以使用 Intent 标志修改默认行为：
    - `FLAG_ACTIVITY_NEW_TASK`
    - `FLAG_ACTIVITY_CLEAR_TOP`
    - `FLAG_ACTIVITY_SINGLE_TOP`

## 六、Activity 的配置变更处理

当屏幕旋转等配置变更时，默认会销毁并重建 Activity,生命周期如下所示：

```
屏幕旋转 → onPause() → onSaveInstanceState() → onStop() → onDestroy() → 销毁状态
开始重建 → onCreate() → onStart() → onRestoreInstanceState() → onResume() → 重建完成
```

如果不想Activity进行重建，可以通过以下方式处理：

1. **保存状态**：
```java
@Override
protected void onSaveInstanceState(Bundle outState) {
    super.onSaveInstanceState(outState);
    outState.putString("key", "value");
}

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    if (savedInstanceState != null) {
        String value = savedInstanceState.getString("key");
    }
}
```

2. **固定配置**：
   在 AndroidManifest.xml 中：
```xml
<activity android:name=".MainActivity"
    android:configChanges="orientation|screenSize|keyboardHidden" />
```

然后重写：
```java
@Override
public void onConfigurationChanged(Configuration newConfig) {
    super.onConfigurationChanged(newConfig);
    // 手动处理配置变更
}
```

禁止Activity重建，之后的生命周期如下：

```
屏幕开始旋转 → onPause() → onConfigurationChanged() → onResume() → 屏幕旋转完成
```

## 七、Activity 的最佳实践

1. **单一职责**：每个 Activity 应专注于单一功能
2. **合理使用生命周期**：在正确的方法中执行相应操作
3. **内存管理**：避免在 Activity 中保存大量数据
4. **响应速度**：onCreate 中避免耗时操作
5. **状态保存**：妥善处理配置变更时的状态保存

## 八、常见问题与解决方案

1. **内存泄漏**：
    - 避免在静态变量或单例中持有 Activity 引用
    - 使用 WeakReference 如果需要持有 Context

2. **生命周期混乱**：
    - 使用 ViewModel 保存 UI 相关数据
    - 使用 LiveData 观察数据变化

3. **启动速度慢**：
    - 减少 onCreate 中的工作量
    - 使用 Splash Screen

4. **多窗口模式适配**：
    - 正确处理配置变更
    - 考虑分屏和画中画模式下的用户体验

Activity 是 Android 开发的基础，深入理解其工作原理对于构建高质量应用至关重要。随着 Android 的发展，Jetpack 组件如 ViewModel 和 LiveData 可以帮助更好地管理 Activity 相关的数据和生命周期。

