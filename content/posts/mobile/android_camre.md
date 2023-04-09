---
title: "Android自定义拍照实现"
date: 2023-04-07T10:30:23+08:00
tags: ["Android"]
categories: [ "移动端"]
draft: false
---
###  前言
由于网上大部分自定义相机的实现，都是耦合性比较强的，不方便今后的复用，所以我自己实现了一套自定义相机，方便以后的扩展。自定义相机分为以下3个部分。
*  相机的预览布局SurfaceView ，方便用户实时预览。写成自定义控件，方便今后的复用。
*  相机的自动聚焦以及点触聚焦，拍照需要聚焦，要不然拍出的图片很可能是模糊的。写成自定义控件，方便今后的复用。
*  相机的自定义布局，这部分随着需求的迭代变换，前面的2大块不需要改动。
   ![](/images/camre_module.webp)
   


### 一、预览布局的实现
###### （1） 抽离预览图层为一个单独的自定义控件CameraPreview ，传递Camre对象，设置必要的相机预览参数。
```java
package com.focustech.xyz.baselibrary.camera;

import android.app.Activity;
import android.content.Context;
import android.graphics.ImageFormat;
import android.hardware.Camera;
import android.view.SurfaceHolder;
import android.view.SurfaceView;

import com.focustech.xyz.baselibrary.common.XyzLogger;

import java.io.IOException;
import java.util.SortedSet;

/**
 * @author 郭翰林
 * @date 2019/2/28 0028 17:06
 * 注释:相机预览视图
 */
public class CameraPreview extends SurfaceView implements SurfaceHolder.Callback {
    private SurfaceHolder mHolder;
    private Camera mCamera;
    private boolean isPreview;
    private Context context;
    /**
     * 预览尺寸集合
     */
    private final SizeMap mPreviewSizes = new SizeMap();
    /**
     * 图片尺寸集合
     */
    private final SizeMap mPictureSizes = new SizeMap();
    /**
     * 屏幕旋转显示角度
     */
    private int mDisplayOrientation;
    /**
     * 设备屏宽比
     */
    private AspectRatio mAspectRatio;

    /**
     * 注释：构造函数
     * 时间：2019/2/28 0028 17:10
     * 作者：郭翰林
     *
     * @param context
     * @param mCamera
     */
    public CameraPreview(Context context, Camera mCamera) {
        super(context);
        this.context = context;
        this.mCamera = mCamera;
        this.mHolder = getHolder();
        this.mHolder.addCallback(this);
        mHolder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);
        mDisplayOrientation = ((Activity) context).getWindowManager().getDefaultDisplay().getRotation();
        mAspectRatio = AspectRatio.of(16, 9);
    }


    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        try {
            //设置设备高宽比
            mAspectRatio = getDeviceAspectRatio((Activity) context);
            //设置预览方向
            mCamera.setDisplayOrientation(90);
            Camera.Parameters parameters = mCamera.getParameters();
            //获取所有支持的预览尺寸
            mPreviewSizes.clear();
            for (Camera.Size size : parameters.getSupportedPreviewSizes()) {
                mPreviewSizes.add(new Size(size.width, size.height));
            }
            //获取所有支持的图片尺寸
            mPictureSizes.clear();
            for (Camera.Size size : parameters.getSupportedPictureSizes()) {
                mPictureSizes.add(new Size(size.width, size.height));
            }
            Size previewSize = chooseOptimalSize(mPreviewSizes.sizes(mAspectRatio));
            Size pictureSize = mPictureSizes.sizes(mAspectRatio).last();
            //设置相机参数
            parameters.setPreviewSize(previewSize.getWidth(), previewSize.getHeight());
            parameters.setPictureSize(pictureSize.getWidth(), pictureSize.getHeight());
            parameters.setPictureFormat(ImageFormat.JPEG);
            parameters.setRotation(90);
            mCamera.setParameters(parameters);
            //把这个预览效果展示在SurfaceView上面
            mCamera.setPreviewDisplay(holder);
            //开启预览效果
            mCamera.startPreview();
            isPreview = true;
        } catch (IOException e) {
            XyzLogger.e("CameraPreview", "相机预览错误: " + e.getMessage());
        }
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
        if (holder.getSurface() == null) {
            return;
        }
        //停止预览效果
        mCamera.stopPreview();
        //重新设置预览效果
        try {
            mCamera.setPreviewDisplay(mHolder);
        } catch (IOException e) {
            e.printStackTrace();
        }
        mCamera.startPreview();
    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        if (mCamera != null) {
            if (isPreview) {
                //正在预览
                mCamera.stopPreview();
                mCamera.release();
            }
        }
    }


    /**
     * 注释：获取设备屏宽比
     * 时间：2019/3/4 0004 12:55
     * 作者：郭翰林
     */
    private AspectRatio getDeviceAspectRatio(Activity activity) {
        int width = activity.getWindow().getDecorView().getWidth();
        int height = activity.getWindow().getDecorView().getHeight();
        return AspectRatio.of(height, width);
    }

    /**
     * 注释：选择合适的预览尺寸
     * 时间：2019/3/4 0004 11:25
     * 作者：郭翰林
     *
     * @param sizes
     * @return
     */
    @SuppressWarnings("SuspiciousNameCombination")
    private Size chooseOptimalSize(SortedSet<Size> sizes) {
        int desiredWidth;
        int desiredHeight;
        final int surfaceWidth = getWidth();
        final int surfaceHeight = getHeight();
        if (isLandscape(mDisplayOrientation)) {
            desiredWidth = surfaceHeight;
            desiredHeight = surfaceWidth;
        } else {
            desiredWidth = surfaceWidth;
            desiredHeight = surfaceHeight;
        }
        Size result = null;
        for (Size size : sizes) {
            if (desiredWidth <= size.getWidth() && desiredHeight <= size.getHeight()) {
                return size;
            }
            result = size;
        }
        return result;
    }

    /**
     * Test if the supplied orientation is in landscape.
     *
     * @param orientationDegrees Orientation in degrees (0,90,180,270)
     * @return True if in landscape, false if portrait
     */
    private boolean isLandscape(int orientationDegrees) {
        return (orientationDegrees == 90 ||
                orientationDegrees == 270);
    }
}

```
这里有2处地方需要注意，相机要设置正确的预览尺寸和正确的图片的尺寸。如果预览尺寸设置错误，则预览布局会被拉伸或者收缩。如果图片尺寸设置错误，部分机型会导致闪退或者拍出的照片很不清晰。

这里适配预览尺寸和图片尺寸，是根据设备的屏宽比和Carme拿到的说支持的预览尺寸和图片尺寸计算得出应有的预览尺寸和图片尺寸，代码如下。

```java
//设置设备高宽比
mAspectRatio = getDeviceAspectRatio((Activity) context);.

Camera.Parameters parameters = mCamera.getParameters();
//获取所有支持的预览尺寸
mPreviewSizes.clear();
for (Camera.Size size : parameters.getSupportedPreviewSizes()) {
    mPreviewSizes.add(new Size(size.width, size.height));
}
//获取所有支持的图片尺寸
mPictureSizes.clear();
for (Camera.Size size : parameters.getSupportedPictureSizes()) {
    mPictureSizes.add(new Size(size.width, size.height));
}
Size previewSize = chooseOptimalSize(mPreviewSizes.sizes(mAspectRatio));
Size pictureSize = mPictureSizes.sizes(mAspectRatio).last();

//设置相机参数
parameters.setPreviewSize(previewSize.getWidth(), previewSize.getHeight());
parameters.setPictureSize(pictureSize.getWidth(), pictureSize.getHeight());
```
```java
/**
 * 注释：获取设备屏宽比
 * 时间：2019/3/4 0004 12:55
 * 作者：郭翰林
 */
private AspectRatio getDeviceAspectRatio(Activity activity) {
    int width = activity.getWindow().getDecorView().getWidth();
    int height = activity.getWindow().getDecorView().getHeight();
    return AspectRatio.of(height, width);
}
```
```java
/**
 * 注释：选择合适的预览尺寸
 * 时间：2019/3/4 0004 11:25
 * 作者：郭翰林
 *
 * @param sizes
 * @return
 */
@SuppressWarnings("SuspiciousNameCombination")
private Size chooseOptimalSize(SortedSet<Size> sizes) {
    int desiredWidth;
    int desiredHeight;
    final int surfaceWidth = getWidth();
    final int surfaceHeight = getHeight();
    if (isLandscape(mDisplayOrientation)) {
        desiredWidth = surfaceHeight;
        desiredHeight = surfaceWidth;
    } else {
        desiredWidth = surfaceWidth;
        desiredHeight = surfaceHeight;
    }
    Size result = null;
    for (Size size : sizes) {
        if (desiredWidth <= size.getWidth() && desiredHeight <= size.getHeight()) {
            return size;
        }
        result = size;
    }
    return result;
}
```
###### （2）屏宽比AspectRatio的实现
```java
package com.focustech.xyz.baselibrary.camera;

import android.os.Parcel;
import android.os.Parcelable;
import android.support.annotation.NonNull;
import android.support.v4.util.SparseArrayCompat;

/**
 * @author 郭翰林
 * @date 2019/3/4 0004 11:11
 * 注释:屏宽比
 */
public class AspectRatio implements Comparable<AspectRatio>, Parcelable {
    private final static SparseArrayCompat<SparseArrayCompat<AspectRatio>> sCache
            = new SparseArrayCompat<>(16);

    private final int mX;
    private final int mY;

    /**
     * Returns an instance of {@link AspectRatio} specified by {@code x} and {@code y} values.
     * The values {@code x} and {@code} will be reduced by their greatest common divider.
     *
     * @param x The width
     * @param y The height
     * @return An instance of {@link AspectRatio}
     */
    public static AspectRatio of(int x, int y) {
        int gcd = gcd(x, y);
        x /= gcd;
        y /= gcd;
        SparseArrayCompat<AspectRatio> arrayX = sCache.get(x);
        if (arrayX == null) {
            AspectRatio ratio = new AspectRatio(x, y);
            arrayX = new SparseArrayCompat<>();
            arrayX.put(y, ratio);
            sCache.put(x, arrayX);
            return ratio;
        } else {
            AspectRatio ratio = arrayX.get(y);
            if (ratio == null) {
                ratio = new AspectRatio(x, y);
                arrayX.put(y, ratio);
            }
            return ratio;
        }
    }

    /**
     * Parse an {@link AspectRatio} from a {@link String} formatted like "4:3".
     *
     * @param s The string representation of the aspect ratio
     * @return The aspect ratio
     * @throws IllegalArgumentException when the format is incorrect.
     */
    public static AspectRatio parse(String s) {
        int position = s.indexOf(':');
        if (position == -1) {
            throw new IllegalArgumentException("Malformed aspect ratio: " + s);
        }
        try {
            int x = Integer.parseInt(s.substring(0, position));
            int y = Integer.parseInt(s.substring(position + 1));
            return AspectRatio.of(x, y);
        } catch (NumberFormatException e) {
            throw new IllegalArgumentException("Malformed aspect ratio: " + s, e);
        }
    }

    private AspectRatio(int x, int y) {
        mX = x;
        mY = y;
    }

    public int getX() {
        return mX;
    }

    public int getY() {
        return mY;
    }

    public boolean matches(Size size) {
        int gcd = gcd(size.getWidth(), size.getHeight());
        int x = size.getWidth() / gcd;
        int y = size.getHeight() / gcd;
        return mX == x && mY == y;
    }

    @Override
    public boolean equals(Object o) {
        if (o == null) {
            return false;
        }
        if (this == o) {
            return true;
        }
        if (o instanceof AspectRatio) {
            AspectRatio ratio = (AspectRatio) o;
            return mX == ratio.mX && mY == ratio.mY;
        }
        return false;
    }

    @Override
    public String toString() {
        return mX + ":" + mY;
    }

    public float toFloat() {
        return (float) mX / mY;
    }

    @Override
    public int hashCode() {
        // assuming most sizes are <2^16, doing a rotate will give us perfect hashing
        return mY ^ ((mX << (Integer.SIZE / 2)) | (mX >>> (Integer.SIZE / 2)));
    }

    @Override
    public int compareTo(@NonNull AspectRatio another) {
        if (equals(another)) {
            return 0;
        } else if (toFloat() - another.toFloat() > 0) {
            return 1;
        }
        return -1;
    }

    /**
     * @return The inverse of this {@link AspectRatio}.
     */
    public AspectRatio inverse() {
        //noinspection SuspiciousNameCombination
        return AspectRatio.of(mY, mX);
    }

    private static int gcd(int a, int b) {
        while (b != 0) {
            int c = b;
            b = a % b;
            a = c;
        }
        return a;
    }

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeInt(mX);
        dest.writeInt(mY);
    }

    public static final Parcelable.Creator<AspectRatio> CREATOR
            = new Parcelable.Creator<AspectRatio>() {

        @Override
        public AspectRatio createFromParcel(Parcel source) {
            int x = source.readInt();
            int y = source.readInt();
            return AspectRatio.of(x, y);
        }

        @Override
        public AspectRatio[] newArray(int size) {
            return new AspectRatio[size];
        }
    };

}

```
```java
/*
 * Copyright (C) 2016 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.focustech.xyz.baselibrary.camera;

import android.support.annotation.NonNull;


/**
 * 注释：尺寸对象
 * 时间：2019/3/4 0004 11:14
 * 作者：郭翰林
 */
public class Size implements Comparable<Size> {

    private final int mWidth;
    private final int mHeight;

    /**
     * Create a new immutable Size instance.
     *
     * @param width  The width of the size, in pixels
     * @param height The height of the size, in pixels
     */
    public Size(int width, int height) {
        mWidth = width;
        mHeight = height;
    }

    public int getWidth() {
        return mWidth;
    }

    public int getHeight() {
        return mHeight;
    }

    @Override
    public boolean equals(Object o) {
        if (o == null) {
            return false;
        }
        if (this == o) {
            return true;
        }
        if (o instanceof Size) {
            Size size = (Size) o;
            return mWidth == size.mWidth && mHeight == size.mHeight;
        }
        return false;
    }

    @Override
    public String toString() {
        return mWidth + "x" + mHeight;
    }

    @Override
    public int hashCode() {
        // assuming most sizes are <2^16, doing a rotate will give us perfect hashing
        return mHeight ^ ((mWidth << (Integer.SIZE / 2)) | (mWidth >>> (Integer.SIZE / 2)));
    }

    @Override
    public int compareTo(@NonNull Size another) {
        return mWidth * mHeight - another.mWidth * another.mHeight;
    }

}
```
```java
package com.focustech.xyz.baselibrary.camera;


import android.support.v4.util.ArrayMap;

import java.util.Set;
import java.util.SortedSet;
import java.util.TreeSet;

/**
 * @author 郭翰林
 * @date 2019/3/4 0004 11:13
 * 注释:尺寸集合
 */
public class SizeMap {

    private final ArrayMap<AspectRatio, SortedSet<Size>> mRatios = new ArrayMap<>();

    /**
     * Add a new {@link Size} to this collection.
     *
     * @param size The size to add.
     * @return {@code true} if it is added, {@code false} if it already exists and is not added.
     */
    public boolean add(Size size) {
        for (AspectRatio ratio : mRatios.keySet()) {
            if (ratio.matches(size)) {
                final SortedSet<Size> sizes = mRatios.get(ratio);
                if (sizes.contains(size)) {
                    return false;
                } else {
                    sizes.add(size);
                    return true;
                }
            }
        }
        // None of the existing ratio matches the provided size; add a new key
        SortedSet<Size> sizes = new TreeSet<>();
        sizes.add(size);
        mRatios.put(AspectRatio.of(size.getWidth(), size.getHeight()), sizes);
        return true;
    }

    /**
     * Removes the specified aspect ratio and all sizes associated with it.
     *
     * @param ratio The aspect ratio to be removed.
     */
    public void remove(AspectRatio ratio) {
        mRatios.remove(ratio);
    }

    Set<AspectRatio> ratios() {
        return mRatios.keySet();
    }

    SortedSet<Size> sizes(AspectRatio ratio) {
        return mRatios.get(ratio);
    }

    void clear() {
        mRatios.clear();
    }

    boolean isEmpty() {
        return mRatios.isEmpty();
    }

}

```
### 二、相机点触自动聚焦并绘制对焦框的实现
###### （1）抽离聚焦框为单独的自定义组件，传递Carma对象和聚焦回调，设置必要的相机参数
```java
package com.focustech.xyz.baselibrary.camera;

import android.content.Context;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.Rect;
import android.hardware.Camera;
import android.support.v7.widget.AppCompatImageView;
import android.util.AttributeSet;
import android.view.WindowManager;

import com.focustech.xyz.baselibrary.common.XyzLogger;

import java.util.ArrayList;
import java.util.List;

/**
 * @author 郭翰林
 * @date 2019/3/1 0001 9:21
 * 注释:对焦框
 */
public class OverCameraView extends AppCompatImageView {
    private Context context;
    //焦点附近设置矩形区域作为对焦区域
    private Rect touchFocusRect;
    private Paint touchFocusPaint;
    //是否正在对焦
    private boolean isFoucuing;

    public OverCameraView(Context context) {
        this(context, null, 0);
    }

    public OverCameraView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public OverCameraView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context);
    }

    private void init(Context context) {
        this.context = context;
        //画笔设置
        touchFocusPaint = new Paint();
        touchFocusPaint.setColor(Color.GREEN);
        touchFocusPaint.setStyle(Paint.Style.STROKE);
        touchFocusPaint.setStrokeWidth(3);
    }

    public boolean isFoucuing() {
        return isFoucuing;
    }

    public void setFoucuing(boolean foucuing) {
        isFoucuing = foucuing;
    }

    /**
     * 注释：对焦并绘制对焦矩形框
     * 时间：2019/3/1 0001 9:28
     * 作者：郭翰林
     *
     * @param camera
     * @param autoFocusCallback
     * @param x
     * @param y
     */
    public void setTouchFoucusRect(Camera camera, Camera.AutoFocusCallback autoFocusCallback, float x, float y) {
        //以焦点为中心，宽度为200的矩形框
        touchFocusRect = new Rect((int) (x - 100), (int) (y - 100), (int) (x + 100), (int) (y + 100));

        //对焦光感区域
        int left = touchFocusRect.left * 2000 / getWindowWidth(context) - 1000;
        int top = touchFocusRect.top * 2000 / getWindowHeight(context) - 1000;
        int right = touchFocusRect.right * 2000 / getWindowWidth(context) - 1000;
        int bottom = touchFocusRect.bottom * 2000 / getWindowHeight(context) - 1000;
        // 如果超出了(-1000,1000)到(1000, 1000)的范围，则会导致相机崩溃
        left = left < -1000 ? -1000 : left;
        top = top < -1000 ? -1000 : top;
        right = right > 1000 ? 1000 : right;
        bottom = bottom > 1000 ? 1000 : bottom;
        final Rect targetFocusRect = new Rect(left, top, right, bottom);

        //对焦
        doTouchFocus(camera, autoFocusCallback, targetFocusRect);
        //刷新界面，调用onDraw(Canvas canvas)函数绘制矩形框
        postInvalidate();
    }

    /**
     * 注释：设置camera参数，并完成对焦
     * 时间：2019/3/1 0001 9:27
     * 作者：郭翰林
     *
     * @param camera
     * @param autoFocusCallback
     * @param tfocusRect
     */
    public void doTouchFocus(Camera camera, Camera.AutoFocusCallback autoFocusCallback, final Rect tfocusRect) {
        if (camera == null || isFoucuing) {
            return;
        }
        try {
            final List<Camera.Area> focusList = new ArrayList<>();
            Camera.Area focusArea = new Camera.Area(tfocusRect, 1000);
            focusList.add(focusArea);

            Camera.Parameters para = camera.getParameters();
            para.setFocusAreas(focusList);
            para.setMeteringAreas(focusList);
            para.setFocusMode(Camera.Parameters.FOCUS_MODE_AUTO);
            camera.cancelAutoFocus();
            camera.setParameters(para);
            camera.autoFocus(autoFocusCallback);
            isFoucuing = true;
        } catch (Exception e) {
            XyzLogger.e("设置相机参数异常", e.getMessage());
        }
    }

    /**
     * 注释：对焦完成后，清除对焦矩形框
     * 时间：2019/3/1 0001 9:28
     * 作者：郭翰林
     */
    public void disDrawTouchFocusRect() {
        //将对焦区域设置为null，刷新界面后对焦框消失
        touchFocusRect = null;
        //刷新界面，调用onDraw(Canvas canvas)函数
        postInvalidate();
    }

    @Override
    protected void onDraw(Canvas canvas) {
        //在画布上绘图，postInvalidate()后自动调用
        drawTouchFocusRect(canvas);
        super.onDraw(canvas);
    }

    /**
     * 获取屏幕高度
     */
    @SuppressWarnings("deprecation")
    public static int getWindowHeight(Context cxt) {
        WindowManager wm = (WindowManager) cxt
                .getSystemService(Context.WINDOW_SERVICE);
        return wm.getDefaultDisplay().getHeight();

    }

    /**
     * 获取屏幕宽度
     */
    @SuppressWarnings("deprecation")
    public static int getWindowWidth(Context cxt) {
        WindowManager wm = (WindowManager) cxt
                .getSystemService(Context.WINDOW_SERVICE);
        return wm.getDefaultDisplay().getWidth();

    }

    private void drawTouchFocusRect(Canvas canvas) {
        if (null != touchFocusRect) {
            //根据对焦区域targetFocusRect，绘制自己想要的对焦框样式，本文在矩形四个角取L形状
            //左下角
            canvas.drawRect(touchFocusRect.left - 2, touchFocusRect.bottom, touchFocusRect.left + 20, touchFocusRect.bottom + 2, touchFocusPaint);
            canvas.drawRect(touchFocusRect.left - 2, touchFocusRect.bottom - 20, touchFocusRect.left, touchFocusRect.bottom, touchFocusPaint);
            //左上角
            canvas.drawRect(touchFocusRect.left - 2, touchFocusRect.top - 2, touchFocusRect.left + 20, touchFocusRect.top, touchFocusPaint);
            canvas.drawRect(touchFocusRect.left - 2, touchFocusRect.top, touchFocusRect.left, touchFocusRect.top + 20, touchFocusPaint);
            //右上角
            canvas.drawRect(touchFocusRect.right - 20, touchFocusRect.top - 2, touchFocusRect.right + 2, touchFocusRect.top, touchFocusPaint);
            canvas.drawRect(touchFocusRect.right, touchFocusRect.top, touchFocusRect.right + 2, touchFocusRect.top + 20, touchFocusPaint);
            //右下角
            canvas.drawRect(touchFocusRect.right - 20, touchFocusRect.bottom, touchFocusRect.right + 2, touchFocusRect.bottom + 2, touchFocusPaint);
            canvas.drawRect(touchFocusRect.right, touchFocusRect.bottom - 20, touchFocusRect.right + 2, touchFocusRect.bottom, touchFocusPaint);
        }
    }
}

```
###### （2）在Activity中的onTouchEvent函数中触发相机聚焦
```java
@Override
public boolean onTouchEvent(MotionEvent event) {
    if (event.getAction() == MotionEvent.ACTION_DOWN) {
        if (!isFoucing) {
            float x = event.getX();
            float y = event.getY();
            isFoucing = true;
            if (mCamera != null && !isTakePhoto) {
                mOverCameraView.setTouchFoucusRect(mCamera, autoFocusCallback, x, y);
            }
            mRunnable = () -> {
                ToastUtil.showToast(this, "自动聚焦超时,请调整合适的位置拍摄！");
                isFoucing = false;
                mOverCameraView.setFoucuing(false);
                mOverCameraView.disDrawTouchFocusRect();
            };
            //设置聚焦超时
            mHandler.postDelayed(mRunnable, 3000);
        }
    }
    return super.onTouchEvent(event);
}
```
```java
/**
 * 注释：自动对焦回调
 * 时间：2019/3/1 0001 10:02
 * 作者：郭翰林
 */
private Camera.AutoFocusCallback autoFocusCallback = new Camera.AutoFocusCallback() {
    @Override
    public void onAutoFocus(boolean success, Camera camera) {
        isFoucing = false;
        mOverCameraView.setFoucuing(false);
        mOverCameraView.disDrawTouchFocusRect();
        //停止聚焦超时回调
        mHandler.removeCallbacks(mRunnable);
    }
};
```
### 三、自定义相机布局
###### （1）自定义相机预览
![](/images/camre_screen_1.webp)

![](/images/camre_screen_2.webp)


###### （2）自定义相机实现代码
```java
package com.focustech.xyz.baselibrary.camera;

import android.app.Activity;
import android.content.Context;
import android.content.Intent;
import android.hardware.Camera;
import android.os.Bundle;
import android.os.Environment;
import android.os.Handler;
import android.support.annotation.Nullable;
import android.support.v7.app.AppCompatActivity;
import android.view.MotionEvent;
import android.view.View;
import android.widget.Button;
import android.widget.FrameLayout;
import android.widget.ImageView;
import android.widget.RelativeLayout;

import com.bumptech.glide.Glide;
import com.focustech.xyz.baselibrary.R;
import com.focustech.xyz.baselibrary.utils.PermissionUtils;
import com.focustech.xyz.baselibrary.utils.ToastUtil;
import com.newland.springdialog.AnimSpring;
import com.yanzhenjie.permission.AndPermission;
import com.yanzhenjie.permission.Permission;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * @author 郭翰林
 * @date 2019/2/28 0028 16:23
 * 注释:Android自定义相机
 */
public class CameraActivity extends AppCompatActivity implements View.OnClickListener {
    public static final String KEY_IMAGE_PATH = "imagePath";
    /**
     * 相机预览
     */
    private FrameLayout mPreviewLayout;
    /**
     * 拍摄按钮视图
     */
    private RelativeLayout mPhotoLayout;
    /**
     * 确定按钮视图
     */
    private RelativeLayout mConfirmLayout;
    /**
     * 闪光灯
     */
    private ImageView mFlashButton;
    /**
     * 拍照按钮
     */
    private ImageView mPhotoButton;
    /**
     * 取消保存按钮
     */
    private ImageView mCancleSaveButton;
    /**
     * 保存按钮
     */
    private ImageView mSaveButton;
    /**
     * 聚焦视图
     */
    private OverCameraView mOverCameraView;
    /**
     * 相机类
     */
    private Camera mCamera;
    /**
     * Handle
     */
    private Handler mHandler = new Handler();
    private Runnable mRunnable;
    /**
     * 取消按钮
     */
    private Button mCancleButton;
    /**
     * 是否开启闪光灯
     */
    private boolean isFlashing;
    /**
     * 图片流暂存
     */
    private byte[] imageData;
    /**
     * 拍照标记
     */
    private boolean isTakePhoto;
    /**
     * 是否正在聚焦
     */
    private boolean isFoucing;
    /**
     * 蒙版类型
     */
    private MongolianLayerType mMongolianLayerType;
    /**
     * 蒙版图片
     */
    private ImageView mMaskImage;
    /**
     * 护照出入境蒙版
     */
    private ImageView mPassportEntryAndExitImage;
    /**
     * 提示文案容器
     */
    private RelativeLayout rlCameraTip;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_camre_layout);
        mMongolianLayerType = (MongolianLayerType) getIntent().getSerializableExtra("MongolianLayerType");
        PermissionUtils.applicationPermissions(this, new PermissionUtils.PermissionListener() {
            @Override
            public void onSuccess(Context context) {
                initView();
                setOnclickListener();
            }

            @Override
            public void onFailed(Context context) {
                if (AndPermission.hasAlwaysDeniedPermission(context, Permission.Group.CAMERA)
                        && AndPermission.hasAlwaysDeniedPermission(context, Permission.Group.STORAGE)) {
                    AndPermission.with(context).runtime().setting().start();
                }
                ToastUtil.showToast(context, context.getString(com.focustech.xyz.baselibrary.R.string.permission_camra_storage));
                finish();
            }
        }, Permission.Group.STORAGE, Permission.Group.CAMERA);
    }

    /**
     * 启动拍照界面
     *
     * @param activity
     * @param requestCode
     * @param type
     */
    public static void startMe(Activity activity, int requestCode, MongolianLayerType type) {
        Intent intent = new Intent(activity, CameraActivity.class);
        intent.putExtra("MongolianLayerType", type);
        activity.startActivityForResult(intent, requestCode);
    }

    /**
     * 注释：获取蒙版图片
     * 时间：2019/3/4 0004 17:19
     * 作者：郭翰林
     *
     * @return
     */
    private int getMaskImage() {
        if (mMongolianLayerType == MongolianLayerType.BANK_CARD) {
            return R.mipmap.bank_card;
        } else if (mMongolianLayerType == MongolianLayerType.HK_MACAO_TAIWAN_PASSES_POSITIVE) {
            return R.mipmap.hk_macao_taiwan_passes_positive;
        } else if (mMongolianLayerType == MongolianLayerType.HK_MACAO_TAIWAN_PASSES_NEGATIVE) {
            return R.mipmap.hk_macao_taiwan_passes_negative;
        } else if (mMongolianLayerType == MongolianLayerType.IDCARD_POSITIVE) {
            return R.mipmap.idcard_positive;
        } else if (mMongolianLayerType == MongolianLayerType.IDCARD_NEGATIVE) {
            return R.mipmap.idcard_negative;
        } else if (mMongolianLayerType == MongolianLayerType.PASSPORT_PERSON_INFO) {
            return R.mipmap.passport_person_info;
        }
        return 0;
    }

    /**
     * 注释：设置监听事件
     * 时间：2019/3/1 0001 11:13
     * 作者：郭翰林
     */
    private void setOnclickListener() {
        mCancleButton.setOnClickListener(this);
        mCancleSaveButton.setOnClickListener(this);
        mFlashButton.setOnClickListener(this);
        mPhotoButton.setOnClickListener(this);
        mSaveButton.setOnClickListener(this);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        if (event.getAction() == MotionEvent.ACTION_DOWN) {
            if (!isFoucing) {
                float x = event.getX();
                float y = event.getY();
                isFoucing = true;
                if (mCamera != null && !isTakePhoto) {
                    mOverCameraView.setTouchFoucusRect(mCamera, autoFocusCallback, x, y);
                }
                mRunnable = () -> {
                    ToastUtil.showToast(this, "自动聚焦超时,请调整合适的位置拍摄！");
                    isFoucing = false;
                    mOverCameraView.setFoucuing(false);
                    mOverCameraView.disDrawTouchFocusRect();
                };
                //设置聚焦超时
                mHandler.postDelayed(mRunnable, 3000);
            }
        }
        return super.onTouchEvent(event);
    }

    /**
     * 注释：自动对焦回调
     * 时间：2019/3/1 0001 10:02
     * 作者：郭翰林
     */
    private Camera.AutoFocusCallback autoFocusCallback = new Camera.AutoFocusCallback() {
        @Override
        public void onAutoFocus(boolean success, Camera camera) {
            isFoucing = false;
            mOverCameraView.setFoucuing(false);
            mOverCameraView.disDrawTouchFocusRect();
            //停止聚焦超时回调
            mHandler.removeCallbacks(mRunnable);
        }
    };

    /**
     * 注释：拍照并保存图片到相册
     * 时间：2019/3/1 0001 15:37
     * 作者：郭翰林
     */
    private void takePhoto() {
        isTakePhoto = true;
        //调用相机拍照
        mCamera.takePicture(null, null, null, (data, camera1) -> {
            //视图动画
            mPhotoLayout.setVisibility(View.GONE);
            mConfirmLayout.setVisibility(View.VISIBLE);
            AnimSpring.getInstance(mConfirmLayout).startRotateAnim(120, 360);
            imageData = data;
            //停止预览
            mCamera.stopPreview();
        });
    }

    /**
     * 注释：切换闪光灯
     * 时间：2019/3/1 0001 15:40
     * 作者：郭翰林
     */
    private void switchFlash() {
        isFlashing = !isFlashing;
        mFlashButton.setImageResource(isFlashing ? R.mipmap.flash_open : R.mipmap.flash_close);
        AnimSpring.getInstance(mFlashButton).startRotateAnim(120, 360);
        try {
            Camera.Parameters parameters = mCamera.getParameters();
            parameters.setFlashMode(isFlashing ? Camera.Parameters.FLASH_MODE_TORCH : Camera.Parameters.FLASH_MODE_OFF);
            mCamera.setParameters(parameters);
        } catch (Exception e) {
            ToastUtil.showToast(this, "该设备不支持闪光灯");
        }
    }

    /**
     * 注释：取消保存
     * 时间：2019/3/1 0001 16:31
     * 作者：郭翰林
     */
    private void cancleSavePhoto() {
        mPhotoLayout.setVisibility(View.VISIBLE);
        mConfirmLayout.setVisibility(View.GONE);
        AnimSpring.getInstance(mPhotoLayout).startRotateAnim(120, 360);
        //开始预览
        mCamera.startPreview();
        imageData = null;
        isTakePhoto = false;
    }

    /**
     * 解析拍出照片的路径
     *
     * @param data
     * @return
     */
    public static String parseResult(Intent data) {
        return data.getStringExtra(KEY_IMAGE_PATH);
    }

    @Override
    public void onClick(View v) {
        int id = v.getId();
        if (id == R.id.cancle_button) {
            finish();
        } else if (id == R.id.take_photo_button) {
            if (!isTakePhoto) {
                takePhoto();
            }
        } else if (id == R.id.flash_button) {
            switchFlash();
        } else if (id == R.id.save_button) {
            savePhoto();
        } else if (id == R.id.cancle_save_button) {
            cancleSavePhoto();
        }
    }


    /**
     * 注释：蒙版类型
     * 时间：2019/2/28 0028 16:26
     * 作者：郭翰林
     */
    public enum MongolianLayerType {
        /**
         * 护照个人信息
         */
        PASSPORT_PERSON_INFO,
        /**
         * 护照出入境
         */
        PASSPORT_ENTRY_AND_EXIT,
        /**
         * 身份证正面
         */
        IDCARD_POSITIVE,
        /**
         * 身份证反面
         */
        IDCARD_NEGATIVE,
        /**
         * 港澳通行证正面
         */
        HK_MACAO_TAIWAN_PASSES_POSITIVE,
        /**
         * 港澳通行证反面
         */
        HK_MACAO_TAIWAN_PASSES_NEGATIVE,
        /**
         * 银行卡
         */
        BANK_CARD
    }

    /**
     * 注释：初始化视图
     * 时间：2019/3/1 0001 11:12
     * 作者：郭翰林
     */
    private void initView() {
        mCancleButton = findViewById(R.id.cancle_button);
        mPreviewLayout = findViewById(R.id.camera_preview_layout);
        mPhotoLayout = findViewById(R.id.ll_photo_layout);
        mConfirmLayout = findViewById(R.id.ll_confirm_layout);
        mPhotoButton = findViewById(R.id.take_photo_button);
        mCancleSaveButton = findViewById(R.id.cancle_save_button);
        mSaveButton = findViewById(R.id.save_button);
        mFlashButton = findViewById(R.id.flash_button);
        mMaskImage = findViewById(R.id.mask_img);
        rlCameraTip = findViewById(R.id.camera_tip);
        mPassportEntryAndExitImage = findViewById(R.id.passport_entry_and_exit_img);

        mCamera = Camera.open();
        CameraPreview preview = new CameraPreview(this, mCamera);
        mOverCameraView = new OverCameraView(this);
        mPreviewLayout.addView(preview);
        mPreviewLayout.addView(mOverCameraView);
        if (mMongolianLayerType == null) {
            mMaskImage.setVisibility(View.GONE);
            rlCameraTip.setVisibility(View.GONE);
            return;
        }
        //设置蒙版,护照出入境蒙版特殊处理
        if (mMongolianLayerType != MongolianLayerType.PASSPORT_ENTRY_AND_EXIT) {
            Glide.with(this).load(getMaskImage()).into(mMaskImage);
        } else {
            mMaskImage.setVisibility(View.GONE);
            mPassportEntryAndExitImage.setVisibility(View.VISIBLE);
        }
    }

    /**
     * 注释：保持图片
     * 时间：2019/3/1 0001 16:32
     * 作者：郭翰林
     */
    private void savePhoto() {
        FileOutputStream fos = null;
        String cameraPath = Environment.getExternalStorageDirectory().getPath() + File.separator + "DCIM" + File.separator + "Camera";
        //相册文件夹
        File cameraFolder = new File(cameraPath);
        if (!cameraFolder.exists()) {
            cameraFolder.mkdirs();
        }
        //保存的图片文件
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyyMMdd_HHmmss");
        String imagePath = cameraFolder.getAbsolutePath() + File.separator + "IMG_" + simpleDateFormat.format(new Date()) + ".jpg";
        File imageFile = new File(imagePath);
        try {
            fos = new FileOutputStream(imageFile);
            fos.write(imageData);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (fos != null) {
                try {
                    fos.close();
                    Intent intent = new Intent();
                    intent.putExtra(KEY_IMAGE_PATH, imagePath);
                    setResult(RESULT_OK, intent);
                } catch (IOException e) {
                    setResult(RESULT_FIRST_USER);
                    e.printStackTrace();
                }
            }
            finish();
        }
    }
}

```
Activity自定义布局 `R.layout.activity_camre_layout`
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!--相机预览视图-->
    <FrameLayout
        android:id="@+id/camera_preview_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"></FrameLayout>
    <!--蒙版区域-->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:orientation="vertical">
        <!--提示文字-->
        <RelativeLayout
            android:id="@+id/camera_tip"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_centerInParent="true"
                android:background="@drawable/tip_layout_shape"
                android:gravity="center"
                android:text="请参照辅助线进行拍摄"
                android:textColor="#fff"
                android:textSize="12sp" />
        </RelativeLayout>

        <!--蒙版图片-->
        <ImageView
            android:id="@+id/mask_img"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginLeft="20dp"
            android:layout_marginTop="25dp"
            android:layout_marginRight="20dp"
            android:scaleType="fitCenter"
            android:src="@mipmap/hk_macao_taiwan_passes_positive"
            android:visibility="visible" />

        <ImageView
            android:id="@+id/passport_entry_and_exit_img"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginLeft="54dp"
            android:layout_marginTop="20dp"
            android:layout_marginRight="54dp"
            android:layout_marginBottom="50dp"
            android:scaleType="fitCenter"
            android:src="@mipmap/passport_entry_and_exit"
            android:visibility="gone" />


    </LinearLayout>
    <!--顶部视图-->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="80dp"
        android:gravity="bottom"
        android:orientation="horizontal"
        android:padding="15dp">

        <ImageView
            android:id="@+id/flash_button"
            android:layout_width="25dp"
            android:layout_height="25dp"
            android:src="@mipmap/flash_close" />
    </LinearLayout>

    <!--拍照完成确定视图-->
    <RelativeLayout
        android:id="@+id/ll_confirm_layout"
        android:layout_width="match_parent"
        android:layout_height="150dp"
        android:layout_alignParentBottom="true"
        android:padding="50dp"
        android:visibility="gone">

        <ImageView
            android:id="@+id/cancle_save_button"
            android:layout_width="50dp"
            android:layout_height="50dp"
            android:layout_centerVertical="true"
            android:src="@mipmap/failed" />

        <ImageView
            android:id="@+id/save_button"
            android:layout_width="50dp"
            android:layout_height="50dp"
            android:layout_alignParentRight="true"
            android:layout_centerVertical="true"
            android:src="@mipmap/success" />
    </RelativeLayout>

    <!--底部拍照按钮-->
    <RelativeLayout
        android:id="@+id/ll_photo_layout"
        android:layout_width="match_parent"
        android:layout_height="150dp"
        android:layout_alignParentBottom="true"
        android:padding="15dp"
        android:visibility="visible">

        <Button
            android:id="@+id/cancle_button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerVertical="true"
            android:background="@null"
            android:text="取消"
            android:textColor="#fff"
            android:textSize="14sp" />

        <ImageView
            android:id="@+id/take_photo_button"
            android:layout_width="80dp"
            android:layout_height="80dp"
            android:layout_centerInParent="true"
            android:src="@mipmap/take_button" />
    </RelativeLayout>


</RelativeLayout>
```
###  四、Demo链接
### 欢迎Star
### GitHub:[https://github.com/RmondJone/AndroidCamera](https://github.com/RmondJone/AndroidCamera)




