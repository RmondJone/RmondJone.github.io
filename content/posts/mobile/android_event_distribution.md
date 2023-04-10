---
title: "Android 事件分发机制"
date: 2023-04-10T15:33:22+08:00
draft: false
categories: ["移动端"]
tags: ["Android"]
---

**事件的下发**

当点击事件产生后会由 Activity 来处理，传递给 PhoneWindow，再传递给DecorView，最后传递给顶层的ViewGroup。

* **事件在ViewGroup中的处理**
  首先会走进ViewGroup中的dispatchTouchEvent方法，dispatchTouchEvent方法中调用onInterceptTouchEvent方法判断是否拦截，如果拦截则交给自身的onTouchEvent方法处理。如果放回false表示不拦截，事件下发给子视图，如此反复。

* **事件在View中的处理**
  如果传递给底层的View， View是没有子View的，就会调用View的dispatchTouchEvent方法，一般情况下最终会调用View的 onTouchEvent方法。

![](/images/android_event.webp)


**事件的处理**

如果处理事件的视图的dispatchTouchEvent方法或者onTouchEvent()方法返回true,则表示事件已被消费，流程中止。如果为false表示事件并未被处理，那么此时就会调用父视图的onTouchEvent()方法，如此反复直到遍历到最顶层的Activity。

**QA**

如果一个Button放在一个LinearLayout中，这个时候手指从Button上按住然后滑动到外层会不会触发Button的点击事件？

答：不会，因为点击事件最后都会在View的onTouchEvent中触发performClick()，而只有在手指抬起时，走MotionEvent.ACTION_UP里的处理时，才会走到。而这时手指已经不在Button的处理范围中，所以不会走到MotionEvent.ACTION_UP方法里。

事件|简介
--|--
ACTION_DOWN	|手指 初次接触到屏幕 时触发。
ACTION_MOVE	|手指 在屏幕上滑动 时触发，会多次触发。
ACTION_UP	|手指 离开屏幕 时触发。
ACTION_CANCEL	|事件 被上层拦截 时触发。
ACTION_OUTSIDE	|手指 不在控件区域 时触发。

