---
title: "Android 通知栏基础使用"
date: 2025-04-28T17:06:45+08:00
draft: false
categories: ["移动端"]
tags: ["Android"]
---

# Android 通知栏开发详解

## 一、通知栏基础概念

### 1. 通知栏核心组件
| 组件 | 作用 | 示例 |
|------|------|------|
| NotificationChannel | 通知分类渠道 | 消息、营销、系统通知 |
| NotificationCompat.Builder | 通知构建器 | 设置图标、标题、内容 |
| PendingIntent | 点击行为控制 | 跳转Activity、启动Service |

### 2. 通知优先级变迁
- **Android 7.1及以下**：`PRIORITY_HIGH`等优先级常量
- **Android 8.0+**：通过`NotificationChannel`设置重要性级别

## 二、创建基础通知

### 1. 必须步骤代码
```kotlin
// 1. 创建通知渠道（Android 8.0+必须）
val channelId = "default_channel"
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
    val channel = NotificationChannel(
        channelId,
        "默认通知",
        NotificationManager.IMPORTANCE_DEFAULT
    ).apply {
        description = "普通通知消息"
    }
    getSystemService(NotificationManager::class.java)
        .createNotificationChannel(channel)
}

// 2. 构建通知
val notification = NotificationCompat.Builder(this, channelId)
    .setSmallIcon(R.drawable.ic_notification)
    .setContentTitle("新消息")
    .setContentText("您收到一条新消息")
    .setPriority(NotificationCompat.PRIORITY_DEFAULT)
    .build()

// 3. 显示通知
NotificationManagerCompat.from(this)
    .notify(notificationId, notification)
```

### 2. 通知属性详解
```kotlin
NotificationCompat.Builder(context, CHANNEL_ID).apply {
    setContentTitle("标题")            // 通知标题
    setContentText("内容")            // 短内容
    setStyle(NotificationCompat.BigTextStyle()
        .bigText("长文本内容..."))     // 展开后内容
    setLargeIcon(bitmap)             // 大图标(Bitmap)
    setAutoCancel(true)              // 点击后自动消失
    setColor(ContextCompat.getColor(context, R.color.red)) // 强调色
    setSound(RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION)) // 声音
    setVibrate(longArrayOf(0, 300, 200, 300)) // 振动模式
}
```

## 三、高级通知功能

### 1. 进度条通知
```kotlin
val builder = NotificationCompat.Builder(this, channelId)
    .setContentTitle("文件下载")
    .setContentText("正在下载...")
    .setSmallIcon(R.drawable.ic_download)
    .setProgress(100, progress, false) // 最大值,当前值,是否不确定

// 更新进度
builder.setProgress(100, newProgress, false)
notificationManager.notify(id, builder.build())

// 完成后更新
builder.setContentText("下载完成")
    .setProgress(0, 0, false)
```

### 2. 操作按钮
```kotlin
val replyIntent = Intent(this, ReplyReceiver::class.java)
val replyPendingIntent = PendingIntent.getBroadcast(
    this, 0, replyIntent, PendingIntent.FLAG_UPDATE_CURRENT
)

val notification = NotificationCompat.Builder(this, channelId)
    .addAction(
        R.drawable.ic_reply,
        "回复",
        replyPendingIntent
    )
    .addAction(
        R.drawable.ic_delete,
        "删除",
        getDeletePendingIntent()
    )
```

### 3. 直接回复通知
```kotlin
// Android 7.0+ 支持
val remoteInput = RemoteInput.Builder("reply_key")
    .setLabel("输入回复内容")
    .build()

val action = NotificationCompat.Action.Builder(
    R.drawable.ic_reply,
    "回复",
    replyPendingIntent
).addRemoteInput(remoteInput)
 .build()

NotificationCompat.Builder(this, channelId)
    .addAction(action)
    .setShowWhen(true)
```

## 四、通知样式扩展

### 1. 大图样式
```kotlin
val style = NotificationCompat.BigPictureStyle()
    .bigPicture(bitmap) // 设置大图
    .setBigContentTitle("大图标题")
    .setSummaryText("图片说明")

NotificationCompat.Builder(this, channelId)
    .setStyle(style)
```

### 2. 收件箱样式
```kotlin
val style = NotificationCompat.InboxStyle()
    .addLine("消息1")
    .addLine("消息2")
    .setBigContentTitle("2条新消息")
    .setSummaryText("邮件通知")

NotificationCompat.Builder(this, channelId)
    .setStyle(style)
```

### 3. 媒体控制通知
```kotlin
val mediaStyle = NotificationCompat.MediaStyle()
    .setShowActionsInCompactView(0, 1, 2)

NotificationCompat.Builder(this, channelId)
    .setStyle(mediaStyle)
    .addAction(prevAction) // 上一首
    .addAction(pauseAction) // 暂停
    .addAction(nextAction) // 下一首
```

## 五、最佳实践

### 1. 兼容性处理方案
```kotlin
fun showNotification(context: Context) {
    val builder = NotificationCompat.Builder(context, createChannel(context))
    
    // 处理不同版本差异
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
        builder.setCategory(Notification.CATEGORY_MESSAGE)
               .setVisibility(Notification.VISIBILITY_PUBLIC)
    }
    
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
        builder.setForegroundServiceBehavior(
            Notification.FOREGROUND_SERVICE_IMMEDIATE
        )
    }
}
```

### 2. 通知分组
```kotlin
// Android 7.0+ 支持
val summaryNotification = NotificationCompat.Builder(context, channelId)
    .setContentTitle("3条新消息")
    .setContentText("来自微信群聊")
    .setSmallIcon(R.drawable.ic_group)
    .setGroup("chat_group")
    .setGroupSummary(true)
    .build()

notificationManager.notify(groupId, summaryNotification)
```

### 3. 前台服务通知
```kotlin
class MyService : Service() {
    override fun onCreate() {
        val notification = buildNotification()
        startForeground(notificationId, notification)
    }
    
    private fun buildNotification(): Notification {
        return NotificationCompat.Builder(this, createChannel(this))
            .setContentTitle("服务运行中")
            .setContentText("正在同步数据...")
            .setSmallIcon(R.drawable.ic_sync)
            .build()
    }
}
```

## 六、常见问题解决

### 1. 通知不显示
- 检查渠道是否创建（Android 8.0+）
- 验证通知权限是否开启
- 确保设置了`smallIcon`
- 华为/小米等厂商需要加入自启动白名单

### 2. 点击无响应
```kotlin
// 正确创建PendingIntent
val intent = Intent(this, TargetActivity::class.java).apply {
    flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
}
val pendingIntent = PendingIntent.getActivity(
    this, 0, intent, 
    PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
)
```

### 3. 通知栏图标显示为白色
- 使用纯Alpha通道图标
- 图标背景应为透明
- 推荐尺寸24x24dp（mdpi基准）
