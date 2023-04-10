---
title: "H5页面由于沉浸式状态栏导致的软件盘遮挡问题"
date: 2023-04-10T17:13:16+08:00
draft: false
categories: ["移动端"]
tags: ["Android"]
---

## 一、场景介绍
在APP中前端反馈在内嵌H5页面点击输入框，软件盘遮挡了用户输入的区域，布局无法自动顶起。然后我这边排查了原因，最后排查到时因为设置了沉浸式状态栏导致的。去除了沉浸式状态栏，但是过了几天测试又和我反馈在部分机型上状态栏显示有问题，没有沉浸式，WTF？？？这个就很无语了，那有没有解决方案呢？最后我想到从布局分发入手，解决了此问题

## 二、解决过程
重写LinearLayout的fitSystemWindows方法和onApplyWindowInsets方法，改变子布局的分发流程。然后在这个布局上添加android:fitsSystemWindows="true"来解决软键盘遮挡的问题。代码如下：

```java
package com.twl.qichechaoren_business.libraryweex.h5;

import android.annotation.TargetApi;
import android.content.Context;
import android.graphics.Rect;
import android.os.Build;
import android.util.AttributeSet;
import android.view.WindowInsets;
import android.widget.LinearLayout;

/**
 * 注释：适配沉浸式状态栏软键盘不能被顶起的问题，在此布局上设置fitsSystemWindows=true
 * 时间：2021/5/27 0027 11:57
 * 作者：郭翰林
 */
public class ResizeLinearLayout extends LinearLayout {
    private int[] mInsets = new int[4];

    public ResizeLinearLayout(Context context) {
        super(context);
    }

    public ResizeLinearLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public ResizeLinearLayout(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    public ResizeLinearLayout(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);
    }

    @Override
    protected final boolean fitSystemWindows(Rect insets) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            // Intentionally do not modify the bottom inset. For some reason,
            // if the bottom inset is modified, window resizing stops working.

            mInsets[0] = insets.left;
            mInsets[1] = insets.top;
            mInsets[2] = insets.right;

            insets.left = 0;
            insets.top = 0;
            insets.right = 0;
        }

        return super.fitSystemWindows(insets);
    }

    @Override
    public final WindowInsets onApplyWindowInsets(WindowInsets insets) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT_WATCH) {
            mInsets[0] = insets.getSystemWindowInsetLeft();
            mInsets[1] = insets.getSystemWindowInsetTop();
            mInsets[2] = insets.getSystemWindowInsetRight();
            return super.onApplyWindowInsets(insets.replaceSystemWindowInsets(0, 0, 0,
                    insets.getSystemWindowInsetBottom()));
        } else {
            return insets;
        }
    }
}
```

```xml
    <com.twl.qichechaoren_business.libraryweex.h5.ResizeLinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:fitsSystemWindows="true">

        <WebView
            android:id="@+id/webview"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:overScrollMode="never" />
    </com.twl.qichechaoren_business.libraryweex.h5.ResizeLinearLayout>
```

