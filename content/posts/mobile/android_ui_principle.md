---
title: "Android中UI的绘制流程"
date: 2023-04-10T15:37:24+08:00
draft: false
categories: ["移动端"]
tags: ["Android"]
---
## 前言
我们都知道Android的View的工作流程主要是指measure、layout、draw这三大流程，即测量、布局和绘制，其中measure确定View的测量宽高，layout根据测量的宽高确定View在其父View中的四个顶点的位置，而draw则将View绘制到屏幕上，这样通过ViewGroup的递归遍历，一个View树就展现在屏幕上了。

### Android的视图结构
![](/images/android_ui_1.webp)


### Window的基本概念

Window表示的是一个窗口的概念，它是站在WindowManagerService角度上的一个抽象的概念，Android中所有的视图都是通过Window来呈现的，不管是Activity、Dialog还是Toast，只要有View的地方就一定有Window。

这里需要注意的是，这个抽象的Window概念和PhoneWindow这个类并不是同一个东西，PhoneWindow表示的是手机屏幕的抽象，它充当Activity和DecorView之间的媒介，就算没有PhoneWindow也是可以展示View的。

抛开一切，仅站在WindowManagerService的角度上，Android的界面就是由一个个Window层叠展现的，而Window又是一个抽象的概念，它并不是实际存在的，它是以View的形式存在，这个View就是DecorView。

### DecorView的概念

DecorView是整个Window界面的最顶层View，View的测量、布局、绘制、事件分发都是由DecorView往下遍历这个View树。DecorView作为顶级View，一般情况下它内部会包含一个竖直方向的LinearLayout，在这个LinearLayout里面有上下两个部分(具体情况和Android的版本及主题有关)，上面是【标题栏】，下面是【内容栏】。在Activity中我们通过setContentView所设置的布局文件其实就是被加载到【内容栏】中的，而内容栏的id是content，因此指定布局的方法叫setContent().

![](/images/android_ui_2.webp)


### ViewRoot的概念
ViewRoot对应于ViewRootImpl类，它是连接WindowManager和DecorView的纽带，View的三大流程均是通过ViewRoot来完成的。在ActivityThread中，当Activity对象被创建完之后，会将DecorView添加到Window中，同时会创建对应的ViewRootImpl，并将ViewRootImpl和DecorView建立关联，并保存到WindowManagerGlobal对象中。

View的绘制流程是从ViewRoot的performTraversals方法开始的，它经过measure、layout和draw三个过程才能最终将一个View绘制出来，大致流程如下图：

![](/images/android_ui_3.webp)


## 一、Activity的启动流程
所有Android应用都是由多个Activity和Fragment组成，而一个页面的承载都是由Activity去承载。我们知道了上诉几个概念之后，就可以来分析一下Activity是怎么由创建到UI全部渲染出来的过程。下午是我阅读源码分析得出的Activity时序图，有兴趣的可以自己去看，文章里不再贴代码。

![](/images/android_ui_4.webp)


从上面的时序图，可以得知，渲染的最关键一步是**通过WindowManager实例把之前创建的DecorView实例添加到根视图中**，下面我们详细来看这块的代码的执行过程：

![](/images/android_ui_5.webp)


可以得知Activity最后的渲染是通过ViewRootImpl来实现计算、布局、绘制到屏幕。

## 二、UI的刷新

上一章节，我们知道了Activity的启动到页面的整体UI展现到屏幕上的流程。那么Activity是怎么实现UI页面的刷新的呢？

我们知道Android上的刷新无非3种方法invalidate()、postInvalidate()、requestLayout()，而各种控件最后调用的刷新方式，也是通过这3种方法来实现。

### invalidate()

postInvalidate()和invalidate()区别在于，invalidate()只能UI线程中调用，而postInvalidate()可以在子线程中调用。postInvalidate()最后其实通过ViewRootImpl里的handler切换到UI线程，最终执行invalidate()。invalidate()只会触发视图的onDraw()方法，而不会触发onMeasure()、onLayout()。

所以我们只需要分析invalidate()即可，我们首先来看View中invalidate()的部分源码：

```java
	public void invalidate() {
        invalidate(true);
    }

    public void invalidate(boolean invalidateCache) {
        //invalidateCache 使绘制缓存失效
        invalidateInternal(0, 0, mRight - mLeft, mBottom - mTop, invalidateCache, true);
    }


    void invalidateInternal(int l, int t, int r, int b, boolean invalidateCache,
                            boolean fullInvalidate) {
        ...
        //设置了跳过绘制标记
        if (skipInvalidate()) {
            return;
        }

        //PFLAG_DRAWN 表示此前该View已经绘制过 PFLAG_HAS_BOUNDS表示该View已经layout过，确定过坐标了
        if ((mPrivateFlags & (PFLAG_DRAWN | PFLAG_HAS_BOUNDS)) == (PFLAG_DRAWN | PFLAG_HAS_BOUNDS)
                || (invalidateCache && (mPrivateFlags & PFLAG_DRAWING_CACHE_VALID) == PFLAG_DRAWING_CACHE_VALID)
                || (mPrivateFlags & PFLAG_INVALIDATED) != PFLAG_INVALIDATED
                || (fullInvalidate && isOpaque() != mLastIsOpaque)) {
            if (fullInvalidate) {
                //默认true
                mLastIsOpaque = isOpaque();
                //清除绘制标记
                mPrivateFlags &= ~PFLAG_DRAWN;
            }

            //需要绘制
            mPrivateFlags |= PFLAG_DIRTY;

            if (invalidateCache) {
                //1、加上绘制失效标记
                //2、清除绘制缓存有效标记
                //这两标记在硬件加速绘制分支用到
                mPrivateFlags |= PFLAG_INVALIDATED;
                mPrivateFlags &= ~PFLAG_DRAWING_CACHE_VALID;
            }
            
            final AttachInfo ai = mAttachInfo;
            final ViewParent p = mParent;
            if (p != null && ai != null && l < r && t < b) {
                final Rect damage = ai.mTmpInvalRect;
                //记录需要重新绘制的区域 damge，该区域为该View尺寸
                damage.set(l, t, r, b);
                //p 为该View的父布局
                //调用父布局的invalidateChild
                p.invalidateChild(this, damage);
            }
            ...
        }
    }
```

从上可知，当前要刷新的View确定了刷新区域后即调用了父布局的invalidateChild(xx)方法。该方法为ViewGroup里的final方法。

```java
	public final void invalidateChild(View child, final Rect dirty) {
        final AttachInfo attachInfo = mAttachInfo;
        if (attachInfo != null && attachInfo.mHardwareAccelerated) {
            //1、如果是支持硬件加速，则走该分支
            onDescendantInvalidated(child, child);
            return;
        }
        //2、软件绘制
        ViewParent parent = this;
        if (attachInfo != null) {
            //动画相关，忽略
            ...
            do {
                View view = null;
                if (parent instanceof View) {
                    view = (View) parent;
                }
                ...
                parent = parent.invalidateChildInParent(location, dirty);
                //动画相关
            } while (parent != null);
        }
    }
```
由上可知，在该方法里区分了硬件加速绘制与软件绘制，分别来看看两者区别。

### 硬件加速绘制分支
如果该Window支持硬件加速，则走下边流程：

```java
 	public void onDescendantInvalidated(@NonNull View child, @NonNull View target) {
        mPrivateFlags |= (target.mPrivateFlags & PFLAG_DRAW_ANIMATION);
        
        if ((target.mPrivateFlags & ~PFLAG_DIRTY_MASK) != 0) {
           //此处都会走
            mPrivateFlags = (mPrivateFlags & ~PFLAG_DIRTY_MASK) | PFLAG_DIRTY;
            //清除绘制缓存有效标记
            mPrivateFlags &= ~PFLAG_DRAWING_CACHE_VALID;
        }
        
        if (mLayerType == LAYER_TYPE_SOFTWARE) {
            //如果是开启了软件绘制，则加上绘制失效标记
            mPrivateFlags |= PFLAG_INVALIDATED | PFLAG_DIRTY;
            //更改target指向
            target = this;
        }

        if (mParent != null) {
            //调用父布局的onDescendantInvalidated
            mParent.onDescendantInvalidated(this, target);
        }
    }
```

onDescendantInvalidated 方法的目的是不断向上寻找其父布局，并将父布局PFLAG_DRAWING_CACHE_VALID 标记清空，也就是绘制缓存清空。
而我们知道，根View的mParent指向ViewRootImpl对象，因此来看看它里面的onDescendantInvalidated()方法。

```java
 	@Override
    public void onDescendantInvalidated(@NonNull View child, @NonNull View descendant) {
        // TODO: Re-enable after camera is fixed or consider targetSdk checking this
        // checkThread();
        if ((descendant.mPrivateFlags & PFLAG_DRAW_ANIMATION) != 0) {
            mIsAnimating = true;
        }
        invalidate();
    }

    @UnsupportedAppUsage
    void invalidate() {
        //mDirty 为脏区域，也就是需要重绘的区域
        //mWidth，mHeight 为Window尺寸
        mDirty.set(0, 0, mWidth, mHeight);
        if (!mWillDrawSoon) {
            //开启View 三大流程
            scheduleTraversals();
        }
    }
```

![](/images/android_ui_6.webp)


### 软件绘制分支

如果该Window不支持硬件加速，那么走软件绘制分支：
parent.invalidateChildInParent(location, dirty) 返回mParent，只要mParent不为空那么一直调用invalidateChildInParent(xx)，实际上这也是遍历ViewTree过程，来看看关键invalidateChildInParent(xx)

```java
 public ViewParent invalidateChildInParent(final int[] location, final Rect dirty) {
        //dirty 为失效的区域，也就是需要重绘的区域
        if ((mPrivateFlags & (PFLAG_DRAWN | PFLAG_DRAWING_CACHE_VALID)) != 0) {
            //该View绘制过或者绘制缓存有效
            if ((mGroupFlags & (FLAG_OPTIMIZE_INVALIDATE | FLAG_ANIMATION_DONE))
                    != FLAG_OPTIMIZE_INVALIDATE) {
                //修正重绘的区域
                dirty.offset(location[CHILD_LEFT_INDEX] - mScrollX,
                        location[CHILD_TOP_INDEX] - mScrollY);
                if ((mGroupFlags & FLAG_CLIP_CHILDREN) == 0) {
                    //如果允许子布局超过父布局区域展示
                    //则该dirty 区域需要扩大
                    dirty.union(0, 0, mRight - mLeft, mBottom - mTop);
                }
                final int left = mLeft;
                final int top = mTop;
                if ((mGroupFlags & FLAG_CLIP_CHILDREN) == FLAG_CLIP_CHILDREN) {
                    //默认会走这
                    //如果不允许子布局超过父布局区域展示，则取相交区域
                    if (!dirty.intersect(0, 0, mRight - left, mBottom - top)) {
                        dirty.setEmpty();
                    }
                }
                //记录偏移，用以不断修正重绘区域，使之相对计算出相对屏幕的坐标
                location[CHILD_LEFT_INDEX] = left;
                location[CHILD_TOP_INDEX] = top;
            } else {
                ...
            }
            //标记缓存失效
            mPrivateFlags &= ~PFLAG_DRAWING_CACHE_VALID;
            if (mLayerType != LAYER_TYPE_NONE) {
                //如果设置了缓存类型，则标记该View需要重绘
                mPrivateFlags |= PFLAG_INVALIDATED;
            }
            //返回父布局
            return mParent;
        }
        return null;
    }
```

与硬件加速绘制一致，最终调用ViewRootImpl invalidateChildInParent(xx)，来看看实现

```java
	public ViewParent invalidateChildInParent(int[] location, Rect dirty) {
        checkThread();
        if (DEBUG_DRAW) Log.v(mTag, "Invalidate child: " + dirty);
        if (dirty == null) {
            //脏区域为空，则默认刷新整个窗口
            invalidate();
            return null;
        } else if (dirty.isEmpty() && !mIsAnimating) {
            return null;
        }
        ...
        invalidateRectOnScreen(dirty);
        return null;
    }

    private void invalidateRectOnScreen(Rect dirty) {
        final Rect localDirty = mDirty;
        //合并脏区域，取并集
        localDirty.union(dirty.left, dirty.top, dirty.right, dirty.bottom);
        ...
        if (!mWillDrawSoon && (intersected || mIsAnimating)) {
            //开启View的三大绘制流程
            scheduleTraversals();
        }
    }
```
![](/images/android_ui_7.webp)

### requestLayout()

顾名思义，重新请求布局。来看看View.requestLayout()方法

```java
    public void requestLayout() {
        //清空测量缓存
        if (mMeasureCache != null) mMeasureCache.clear();
        ...
        //添加强制layout 标记，该标记触发layout
        mPrivateFlags |= PFLAG_FORCE_LAYOUT;
        //添加重绘标记
        mPrivateFlags |= PFLAG_INVALIDATED;

        if (mParent != null && !mParent.isLayoutRequested()) {
            //如果上次的layout 请求已经完成
            //父布局继续调用requestLayout
            mParent.requestLayout();
        }
        ...
    }
```
可以看出，这个递归调用和invalidate一样的套路，向上寻找其父布局，一直到ViewRootImpl为止，给每个布局设置PFLAG_FORCE_LAYOUT和PFLAG_INVALIDATED标记。查看ViewRootImpl requestLayout()

```java
    public void requestLayout() {
        //是否正在进行layout过程
        if (!mHandlingLayoutInLayoutRequest) {
            //检查线程是否一致
            checkThread();
            //标记有一次layout的请求
            mLayoutRequested = true;
            //开启View 三大流程
            scheduleTraversals();
        }
    }
```
很明显，requestLayout目的很单纯:

* 1、向上寻找父布局、并设置强制layout标记
* 2、最终开启三大绘制流程

和`invalidate()`一样的配方，当刷新信号来到之时，调用`doTraversal()->performTraversals()`，而在`performTraversals()`里真正执行三大流程。

```java
    private void performTraversals() {
        //mLayoutRequested 在requestLayout时赋值为true
        boolean layoutRequested = mLayoutRequested && (!mStopped || mReportNextDraw);
        if (layoutRequested) {
            //measure 过程
            windowSizeMayChange |= measureHierarchy(host, lp, res,
                    desiredWindowWidth, desiredWindowHeight);
        }
        ...

        final boolean didLayout = layoutRequested && (!mStopped || mReportNextDraw);
        if (didLayout) {
            //layout 过程
            performLayout(lp, mWidth, mHeight);
        }
        ...
    }
```

由此可见：

* 1、requestLayout 最终将会触发Measure、Layout 过程。
* 2、由于没有设置重绘区域，因此Draw 过程将不会触发。

之前设置的PFLAG_FORCE_LAYOUT标记有啥用呢？回忆一下measure 过程：

```java
    public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
        ...
        //requestLayout时，PFLAG_FORCE_LAYOUT 标记被设置
        final boolean forceLayout = (mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT;
        ...
        if (forceLayout || needsLayout) {
            ...
            int cacheIndex = forceLayout ? -1 : mMeasureCache.indexOfKey(key);
            if (cacheIndex < 0 || sIgnoreMeasureCache) {
                //测量
                onMeasure(widthMeasureSpec, heightMeasureSpec);
            } else {
                ...
            }
            ...
            }
        }
    }
```
PFLAG_FORCE_LAYOUT 标记打上之后，会触发onMeasure()测量自身及其子布局。

试想一下，假设View的尺寸改变了，变大了，那么调用了requestLayout后因为走了Measure、Layout 过程，测量、摆放倒是重新设置了，但是不调用Draw出不来效果啊。实际上，View layout时候已经考虑到了。在View.layout(xx)->setFrame(xx)里

```java
    protected boolean setFrame(int left, int top, int right, int bottom) {
        boolean changed = false;

        if (mLeft != left || mRight != right || mTop != top || mBottom != bottom) {
            changed = true;
            ...
            int oldWidth = mRight - mLeft;
            int oldHeight = mBottom - mTop;
            int newWidth = right - left;
            int newHeight = bottom - top;
            boolean sizeChanged = (newWidth != oldWidth) || (newHeight != oldHeight);
            //尺寸发生改变 调用invalidate 传入true，否则传入false
            invalidate(sizeChanged);
            ...
        }
        ...
        return changed;
    }
```
也就是说：

* 1、requestLayout调用后，可能会触发invalidate。
* 2、若是触发了invalidate()，不管传入true还是false，都会走重绘流程。

![](/images/android_ui_8.webp)


**总结**

* invalidate调用后只会触发Draw 过程。
* requestLayout 会触发Measure、Layout过程，如果尺寸发生改变，则会调用invalidate。
* 当涉及View的尺寸、位置变化时使用requestLayout。
* 当仅仅需要重绘时调用invalidate。
* 如果不确定requestLayout 是否触发invalidate，可在requestLayout后继续调用invalidate。

![](/images/android_ui_9.webp)


## 三、自定义ViewGroup的onDraw为什么不走？

由前面的几节内容可以知道，所有的视图的绘制最终都会经过ViewRootImpl来实现，最后都会走到View的onDraw方法里，下面我们详细来看源码：

```java
 public void draw(Canvas canvas) {
        final int privateFlags = mPrivateFlags;
        final boolean dirtyOpaque = (privateFlags & PFLAG_DIRTY_MASK) == PFLAG_DIRTY_OPAQUE &&
                (mAttachInfo == null || !mAttachInfo.mIgnoreDirtyState);
        mPrivateFlags = (privateFlags & ~PFLAG_DIRTY_MASK) | PFLAG_DRAWN;

        /*
         * Draw traversal performs several drawing steps which must be executed
         * in the appropriate order:
         *
         *      1. Draw the background
         *      2. If necessary, save the canvas' layers to prepare for fading
         *      3. Draw view's content
         *      4. Draw children
         *      5. If necessary, draw the fading edges and restore layers
         *      6. Draw decorations (scrollbars for instance)
         */

        // Step 1, draw the background, if needed
        int saveCount;

        if (!dirtyOpaque) {
            drawBackground(canvas);
        }

        // skip step 2 & 5 if possible (common case)
        final int viewFlags = mViewFlags;
        boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
        boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
        if (!verticalEdges && !horizontalEdges) {
            // Step 3, draw the content
            if (!dirtyOpaque) onDraw(canvas);//如果有背景色，走onDraw()方法，如果没有背景色，不走onDraw()方法

            // Step 4, draw the children
            dispatchDraw(canvas);

            // Overlay is part of the content and draws beneath Foreground
            if (mOverlay != null && !mOverlay.isEmpty()) {
                mOverlay.getOverlayView().dispatchDraw(canvas);
            }

            // Step 6, draw decorations (foreground, scrollbars)
            onDrawForeground(canvas);

            // we're done...
            return;
        }
        ......
    }
```

从代码的注释中能清楚的看到绘制的顺序:

1、画背景
2、画canvas的图层(非必须)
3、画View自己
4、画子View
5、如果2执行了，这步要回复图层(非必须)

这个并不是重点，下面的这块代码才是重点：
```java
  //如果有背景色，走onDraw()方法，如果没有背景色，不走onDraw()方法
  if (!dirtyOpaque) onDraw(canvas);

  // Step 4, draw the children
  dispatchDraw(canvas);
```

**总结**

1、自定义ViewGroup的onDraw()方法需要设置背景才会调用
2、自定义ViewGroup正常情况下应该实现dispatchDraw()


