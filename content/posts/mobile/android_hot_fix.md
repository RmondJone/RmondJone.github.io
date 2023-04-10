---
title: "Android热修复方案总结"
date: 2023-04-10T17:16:01+08:00
draft: false
categories: ["移动端"]
tags: ["Android"]
---

## 背景

Android热修复方案有很多，我们没有必要去解析每个框架的热修复具体实现。我们只需要掌握热修复的几个基本原理即可。目前Android热修复的技术方案大致可以归类为以下几种：

* **资源热替换**
* **代码热修复**
* **动态库替换**

## 一、资源修复

大部分采用这种修复的框架都参照了Android Studio的Instant Run的功能，通过反编译Instant Run的功能代码可以看到，其主要实现为MonkeyPatcher的monkeyPatchExistingResources方法。

![](/images/hot_fix_1.webp)

通过以上代码可以看出，资源修复的方法其实很简单，主要分为以下2个步骤：

* 创建新的AssetManager实例，利用反射加载外部(SD卡)需要热修复的资源路径
* 通过当前Activity实例获取Resouces资源类，然后反射得到mAssets属性，把之前新建的AssetManager实例进行动态替换

## 二、代码热修复

代码热修复方案可以归纳为以下3种:类加载方案、底层替换方案、Instant Run方案

### 类加载方案

如果你清楚Android的类加载机制，你一定知道DexPathList.java这个类，本节类加载方案主要就是**基于DexPathList.java的findClass方法**处理逻辑再利用**类加载机制的双亲委托模型**来实现Bug类的动态修复
![](/images/hot_fix_2.webp)

我们先来看DexPathList.java的内部实现逻辑
![](/images/hot_fix_3.webp)

可以看到这里主要就是遍历Element元素，Element内部封装了DexFile，**所以这里可以切入的Hook点**。

我们看一下采用这套方案的流程图是什么样子，假设我们现在有一个Key类需要修复，我打出补丁Patch.dex并且把这个补丁加载放在Elements集合里的第一个。那么加载时序图就如下所示：
![](/images/hot_fix_4.webp)
可以看到我们的Patch.dex里的Key修复类被第一个加载，后面如果还是有Key类的class，根据类加载机制的双亲委托模型，后面的这个Key.class将不会被再次加载。从而实现了Key类的修复。

但也可以看出这个方案明显的劣势，就是**只有应用被重启之后才会生效**，并不能实现立马就生效的效果。

### 底层替换方案
与类加载方案不同的是，底层替换方案不会再次加载新类，而是直接在Native层修改原有的类，使其功能立即生效。

拿方法替换来说，我们的方法在ART虚拟机中都对应着一个ArtMethod结构体。
![](/images/hot_fix_5.webp)
注释1和2，他们是方法的执行入口，当我们调用一个类的方法时就会拿到这个执行入口。通过执行入口我们就可以跳过执行原有逻辑执行我们想要修复的逻辑。

底层替换分为2个主流方案：
* 替换ArtMethod结构体里的注释1和2的字段（兼容性不好，厂商会改）
* 替换整个ArtMethod结构体（兼容性强）

### Instant Run方案
这个也是借鉴了Instant Run里的**ASM动态注入技术**。什么是ASM？ASM是一个Java字节码操控框架，它能够动态的生成类或者增强类的实现。ASM可以直接生成class文件，也可以在类被加载到虚拟机之前动态的改变类的行为。

## 三、动态库的替换
Android平台动态链接库主要指的就是so文件。那也不难理解这类的热更新主要就是重新加载替换的so库。**主要用到了System的load和loadLibrary方法**