---
title: "Android性能优化-绘制篇"
date: 2023-04-10T16:51:36+08:00
draft: false
categories: ["移动端"]
tags: ["Android","性能优化"]
---

## 前言
在日常开发中，Android的性能优化是我们需要一直关注的点。那么本文也是老生常谈，说说Android的性能优化绘制篇。我们在日常开发中该怎么去做绘制优化的分析，以及一些绘制优化的一些场景。

## 一、Android绘制优化可以切入的点

我们知道，打游戏有一个参数叫**fps，也就是帧率，也就是1s内页面刷新了多少次**。如果帧率低于60fps，人的肉眼可以明显感知到画面卡顿。那么要想人眼感觉不卡，一帧绘制的时间也就必须低于1/60s,也就是16.33ms。

那么**Android绘制优化**也就是着手与解决**哪些场景**会导致一帧绘制的时间大于16.33ms。以及有什么工具可以帮助我们快速的发现帧绘制时间异常的地方。一般导致帧绘制时间过长的原因有以下几点：

* **Layout布局过于复杂**
  
  Layout过于复杂，导致绘制时间变长，也就导致了一帧绘制的时间大于16.33ms
  
* **执行动画过多，导致GPU和CPU负载严重**
  
  执行了过多的动画，导致GPU和CPU负载严重，没法全部投入到帧的绘制中，导致绘制时间大于16.33ms
  
* **View的过度绘制**
  
  同一时间某些像素点被绘制了多次，浪费了CPU和GPU资源
  
* **频繁的内存GC**
  
  我们都知道GC是需要暂停的，只有暂停了才能知道哪些对象是可回收的对象。而一般的GC暂停时间都在2ms左右有的时候甚至可能6ms，如果一帧内GC过于频繁，势必导致一帧的绘制的时间大于16.33ms

* **在UI线程中做了耗时操作**

  这个应该不用多说，如果UI线程里做了耗时操作，势必阻塞UI线程，从而影响了帧的绘制

## 二、Android绘制优化的一些工具

* **开发者选项里的查看绘制时长的工具**

  一般手机系统里都会有一个开发者工具，这个开发者工具里有一个查看帧绘制的工具。如下图所示，一帧的绘制时间就是一个柱状，大于红线就说明一帧绘制时间大于16.33ms。

  ![](/images/android_optimization_1.webp)

* **开发者选项里查看过度绘制的工具**

  一般手机系统里都会有一个开发者工具，这个开发者工具里有一个查看过度绘制的工具。如下图所示，如果区域显示红色，则代表该区域过度绘制可以进行优化。

  ![](/images/android_optimization_2.webp)

* **BlockCanary三方检测库**

  [Github仓库地址](https://github.com/seiginonakama/BlockCanaryEx)
 
  BlockCanary 是一款用于 检测 Android 应用卡顿（主线程阻塞） 的开源工具，它能帮助开发者快速定位导致界面卡顿的代码。

  简单来说： 它像一个 “卡顿监控器”，专门盯着 App 的主线程（UI 线程）。 如果主线程被某个任务 卡住太久（比如耗时操作），BlockCanary 就会 记录当时的堆栈、CPU、内存等信息，并生成报告，告诉开发者 哪里出了问题。

  ![](/images/block_canary.jpg)

## 三、BlockCanary原理

![](/images/block_source.jpeg)

**1. 监控主线程的 Looper 消息队列**

Android 的主线程通过 Looper 循环处理消息（Message），每个消息代表一个任务（如 UI 刷新、点击事件等）。

BlockCanary 通过 替换 Looper 的 Printer（默认用于日志输出的接口），在每条消息执行前后插入监控逻辑：

```java
// 伪代码：替换 Looper 的 Printer
Looper.getMainLooper().setMessageLogging(new Printer() {
    @Override
    public void println(String log) {
        if (log.startsWith(">>>>> Dispatching")) {
            // 记录消息开始时间
            startTime = System.currentTimeMillis();
        } else if (log.startsWith("<<<<< Finished")) {
            // 计算消息处理耗时
            long costTime = System.currentTimeMillis() - startTime;
            if (costTime > 阈值（如 2秒）) {
                // 触发卡顿分析
                analyzeBlock(costTime);
            }
        }
    }
});
```

**2. 卡顿判定与数据采集**

当主线程的某个消息处理时间超过设定的阈值（如 2 秒），BlockCanary 会：
* 抓取当前主线程的堆栈（通过 Thread.getStackTrace()），定位卡顿代码位置。
* 采集系统状态（CPU、内存、I/O 负载等），分析是否因系统资源不足导致卡顿。
* 保存卡顿日志（时间、堆栈、设备信息等）。

**3. 生成可视化报告**

将卡顿信息整理成易读的报告（如弹出通知、生成日志文件），开发者可直接查看：

```
BlockCanary 卡顿报告：
- 卡顿时间：2023-10-01 12:00:00  
- 耗时：2.5 秒  
- 堆栈：
    at com.example.MainActivity.onClick(MainActivity.java:100)
    at android.view.View.performClick(View.java:7000)
- CPU 占用：80%（可能资源紧张）
```