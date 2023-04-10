---
title: "Android UI刷新机制"
date: 2023-04-10T17:06:07+08:00
draft: false
categories: ["移动端"]
tags: ["Android"]
---

## 前言
Android屏幕的刷新包含3个步骤：CPU计算屏幕数据、GPU进一步处理和缓存、最后屏幕（Display）再从缓存中把计算的屏幕数据显示出来

对于 Android 而言，第一个步骤：**CPU 计算屏幕数据**指的也就是 View 树的绘制过程，也就是 Activity 对应的视图树从根布局 DecorView 开始层层遍历每个 View，分别执行测量、布局、绘制三个操作的过程。我们重点分析的也是这个步骤，关于后续的2个步骤我们可以理解为底层处理，没必要过于深入。

## Android的UI刷新图解
我们知道Android每隔16.6ms会发送一次垂直同步VSync信息量，1S也就是60帧的画面。下面这个图蓝色的是CPU计算屏幕数据时间戳，绿色的是GPU的处理，最后黄色的是屏幕。我们可以清楚的看到，每帧的画面都会提前一帧去计算以及GPU处理。

![](/images/ui_refresh_1.webp)


如果我们保持页面静止，那么Android还是会16.6ms发送一次垂直同步信号量，App这个时候接受不到屏幕刷新的信号。所以也就不会让 CPU 去计算下一帧画面数据，但是底层仍然会以固定的频率来切换每一帧的画面，只是它后面切换的每一帧画面都一样，所以给我们的感觉就是屏幕没刷新

![](/images/ui_refresh_2.webp)


## Android的UI刷新代码层级理解
我们都知道Android的刷新离不开ViewRootImpl,在上一篇文章[《Android中UI的绘制流程》](/posts/mobile/android_ui_principle)中，大致阐述了Android的UI刷新流程。这里我们进一步深入的理解源码，以及刷新UI的详细流程。首先看图：

![](/images/ui_refresh_3.webp)


* 每一次的View刷新流程都会不停的往上找父视图，直至到ViewRootImpl为止，然后调用ViewRootImpl的sheduleTraversals()方法。

* sheduleTraversals()方法会将遍历绘制 View 树的操作 performTraversals() 封装到 Runnable 里，线程变量Choreographer内部维护一个mCallbackQueue类似于消息队列的阻塞队列，然后调用Choreographer的postCallBack()传递到mCallbackQueue队列中。

* scheduleFrameLocked()方法里如果使用垂直同步则最终会调用到nativeScheduleVsync()申请垂直同步信号量，等待Android下一次发送Vsync信息的时候，会通过JNI回调主动触发Choreographer的onVsync()方法，这个方法进而执行doFrame()。如果不使用垂直同步则直接会执行doFrame()函数。

* doFrame()函数中最后会回到ViewRootImpl里并执行performTraversals()方法，进而执行performMeasure()、performLayout()、performDraw()实现UI的刷新。









