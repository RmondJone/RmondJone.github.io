---
title: "Android Glide框架使用以及原理"
date: 2025-04-24T15:48:24+08:00
draft: false
categories: ["移动端"]
tags: ["Android"]
---

## **一、Android Glide 使用指南**

Glide 是 Android 开发中最常用的图片加载库之一，具有 **高效缓存、内存优化、链式调用** 等特性，适用于加载网络图片、本地图片、GIF 等。以下是 Glide 的基本用法和高级功能。

### **1. 基本用法**
#### **1.1 添加依赖**
在 `build.gradle` 中添加：
```gradle
implementation 'com.github.bumptech.glide:glide:4.16.0'
annotationProcessor 'com.github.bumptech.glide:compiler:4.16.0' // 可选，用于生成 GlideApp
```

#### **1.2 加载网络图片**
```java
Glide.with(context)  // Context, Activity, Fragment 或 View
    .load("https://example.com/image.jpg")  // 图片 URL
    .into(imageView);  // 目标 ImageView
```

#### **1.3 加载本地资源**
```java
// 加载 drawable 资源
Glide.with(context)
    .load(R.drawable.my_image)
    .into(imageView);

// 加载本地文件
Glide.with(context)
    .load(new File("/sdcard/image.jpg"))
    .into(imageView);
```

#### **1.4 加载 GIF**
```java
Glide.with(context)
    .asGif()  // 指定加载 GIF
    .load("https://example.com/anim.gif")
    .into(imageView);
```

### **2. 常用配置**
#### **2.1 占位图 & 错误图**
```java
Glide.with(context)
    .load(url)
    .placeholder(R.drawable.placeholder)  // 加载中显示的图片
    .error(R.drawable.error_image)  // 加载失败显示的图片
    .into(imageView);
```

#### **2.2 调整图片大小**
```java
Glide.with(context)
    .load(url)
    .override(300, 200)  // 强制调整为 300x200 像素
    .into(imageView);
```

#### **2.3 缓存策略**
```java
Glide.with(context)
    .load(url)
    .diskCacheStrategy(DiskCacheStrategy.ALL)  // 缓存原始图片 + 转换后的图片
    .into(imageView);
```
可选缓存策略：
- `DiskCacheStrategy.NONE`：不缓存
- `DiskCacheStrategy.DATA`：只缓存原始图片
- `DiskCacheStrategy.RESOURCE`：只缓存转换后的图片
- `DiskCacheStrategy.ALL`：缓存所有（默认）

#### **2.4 图片变换（圆角、圆形裁剪）**
```java
// 圆形裁剪
Glide.with(context)
    .load(url)
    .circleCrop()
    .into(imageView);

// 圆角图片
Glide.with(context)
    .load(url)
    .transform(new RoundedCorners(16))  // 16px 圆角
    .into(imageView);
```

### **3. 高级用法**
#### **3.1 监听加载状态**
```java
Glide.with(context)
    .load(url)
    .listener(new RequestListener<Drawable>() {
        @Override
        public boolean onLoadFailed(@Nullable GlideException e, Object model, 
                                  Target<Drawable> target, boolean isFirstResource) {
            Log.e("Glide", "加载失败", e);
            return false;
        }

        @Override
        public boolean onResourceReady(Drawable resource, Object model, 
                                     Target<Drawable> target, DataSource dataSource, 
                                     boolean isFirstResource) {
            Log.d("Glide", "加载成功");
            return false;
        }
    })
    .into(imageView);
```

#### **3.2 自定义 GlideModule（全局配置）**
在 `AndroidManifest.xml` 中注册：
```xml
<meta-data
    android:name="com.example.MyGlideModule"
    android:value="GlideModule" />
```
自定义 `GlideModule`：
```java
public class MyGlideModule extends AppGlideModule {
    @Override
    public void applyOptions(Context context, GlideBuilder builder) {
        // 设置全局缓存大小
        builder.setMemoryCache(new LruResourceCache(20 * 1024 * 1024)); // 20MB
    }
}
```

#### **3.3 加载缩略图（先加载低分辨率图）**
```java
Glide.with(context)
    .load(url)
    .thumbnail(0.1f)  // 先加载 10% 大小的缩略图
    .into(imageView);
```

### **4. 常见问题**
#### **4.1 Glide 如何清理缓存？**
```java
// 清理内存缓存（UI 线程）
Glide.get(context).clearMemory();

// 清理磁盘缓存（需在子线程执行）
new Thread(() -> Glide.get(context).clearDiskCache()).start();
```

#### **4.2 如何避免 RecyclerView 图片闪烁？**
使用 `Glide` 的 `signature()` 确保缓存唯一性：
```java
Glide.with(context)
    .load(url)
    .signature(new ObjectKey(version))  // 版本变化时刷新缓存
    .into(imageView);
```

#### **4.3 如何加载 Base64 图片？**
```java
String base64 = "data:image/png;base64,...";
Glide.with(context)
    .load(base64)
    .into(imageView);
```

### **5. 总结**
| **功能**          | **Glide 方法**                     |
|-------------------|-----------------------------------|
| 加载网络图片       | `.load(url)`                      |
| 加载本地图片       | `.load(file)` 或 `.load(resId)`   |
| 加载 GIF          | `.asGif()`                        |
| 占位图 & 错误图   | `.placeholder()` / `.error()`     |
| 图片变换          | `.circleCrop()` / `.transform()`  |
| 缓存控制          | `.diskCacheStrategy()`            |
| 监听加载状态      | `.listener()`                     |

Glide 是 Android 图片加载的 **首选库**，适用于大多数场景，能有效优化内存和性能。

## **二、Android Glide 框架原理详解**

Glide 是一个高效、灵活的 Android **图片加载库**，其核心设计围绕 **快速加载、内存优化、缓存管理** 展开。以下是 Glide 的核心工作原理：

###  **1. Glide 的核心架构**
Glide 的架构分为 **4 层**，各层职责明确：
1. **API 层**（`Glide.with().load().into()`）：提供用户友好的链式调用接口。
2. **Engine 层**：负责 **加载、解码、转换、缓存** 图片。
3. **Decode 层**：处理图片解码（如 Bitmap、GIF、WebP）。
4. **Registry 层**：管理组件（如 `ModelLoader`、`ResourceDecoder`），支持扩展。

### **2. 核心流程（图片加载步骤）**
当调用 `Glide.with(context).load(url).into(imageView)` 时，Glide 的执行流程如下：

![](/images/glide_1.png)

#### **2.1 初始化请求**
1. **`Glide.with(context)`**
    - 绑定生命周期（Activity/Fragment），防止内存泄漏。
    - 通过 `RequestManagerRetriever` 获取 `RequestManager`。

2. **`.load(url)`**
    - 解析数据源（URL、File、Resource 等），生成 `RequestBuilder`。

3. **`.into(imageView)`**
    - 创建 `Target`（如 `ImageViewTarget`）并启动加载任务。

#### **2.2 加载与缓存**

![](/images/glide_2.png)

1. **检查内存缓存**
    - 使用 `LruResourceCache`（基于 LRU 算法）查找是否缓存了目标图片。
    - 如果命中，直接返回 `Bitmap` 并显示。

2. **检查磁盘缓存**
    - 如果内存缓存未命中，检查磁盘缓存（`DiskLruCache`）。
    - 磁盘缓存分为：
        - **原始数据缓存**（未解码的图片数据）。
        - **转换后缓存**（如圆角、裁剪后的图片）。

3. **从源加载**
    - 如果缓存未命中，通过 `ModelLoader`（如 `HttpUrlFetcher`）从网络或本地加载数据。

4. **解码与转换**
    - 使用 `ResourceDecoder` 将原始数据（如 InputStream）解码为 `Bitmap` 或 `GIF`。
    - 应用 `Transformation`（如圆角、裁剪）。

5. **缓存结果**
    - 将解码后的 `Bitmap` 存入内存缓存。
    - 可选是否缓存转换后的图片到磁盘。

6. **显示图片**
    - 通过 `ImageViewTarget` 将图片设置到 `ImageView`。

### **3. 关键优化技术**
#### **3.1 三级缓存机制**
| **缓存层级** | **存储内容**                | **特点**                          |
|--------------|----------------------------|-----------------------------------|
| **活动缓存** | 当前正在使用的图片          | 使用 `WeakReference`，避免重复加载。 |
| **内存缓存** | 解码后的 `Bitmap`           | `LruCache` 管理，默认大小由设备内存决定。 |
| **磁盘缓存** | 原始数据或转换后的图片       | `DiskLruCache`，可配置缓存策略。 |

#### **3.2 生命周期管理**
- 通过 `RequestManager` 绑定 `Activity/Fragment` 生命周期。
- 当页面销毁时，自动取消未完成的请求，避免内存泄漏。

#### **3.3 图片复用（Bitmap Pool）**
- 使用 `BitmapPool` 复用已回收的 `Bitmap` 内存，减少 GC 频率。
- 通过 `inBitmap` 重用内存，提升性能。

#### **3.4 智能大小调整**
- 根据 `ImageView` 的尺寸自动调整图片大小，避免加载过大的 `Bitmap`。
- 支持 `override()` 强制指定尺寸。

### **4. 核心组件解析**
| **组件**            | **作用**                                                                 |
|---------------------|--------------------------------------------------------------------------|
| `RequestManager`    | 管理请求的生命周期，绑定到 Activity/Fragment。                           |
| `Engine`            | 调度加载任务，处理缓存逻辑。                                             |
| `ModelLoader`       | 加载不同数据源（如 URL、File、ContentProvider）。                        |
| `ResourceDecoder`   | 解码原始数据（如 InputStream → Bitmap）。                                |
| `Transformation`    | 图片变换（如圆角、裁剪）。                                               |
| `Target`            | 接收加载结果（如 `ImageViewTarget` 用于显示到 `ImageView`）。            |

### **5. 与其他库对比**
| **特性**          | **Glide**                          | **Picasso**               | **Fresco**                |
|-------------------|------------------------------------|---------------------------|---------------------------|
| **缓存机制**      | 内存 + 磁盘 + 活动缓存             | 内存 + 磁盘               | 三级缓存 + 内存映射       |
| **GIF 支持**      | ✔️                                 | ❌                        | ✔️                        |
| **生命周期管理**  | ✔️（自动绑定）                     | ❌（需手动取消）           | ✔️                        |
| **内存优化**      | BitmapPool + 智能尺寸              | 无特殊优化                | 原生内存管理（更省内存）  |
| **扩展性**        | 高（支持自定义 ModelLoader/解码器）| 一般                      | 低（依赖 Facebook 实现）  |

### **6. 总结**
- **Glide 的核心优势**：
    - **高效缓存**：三级缓存减少 IO 和 CPU 开销。
    - **内存优化**：`BitmapPool` 复用 + 智能尺寸调整。
    - **生命周期集成**：自动管理请求，避免泄漏。
    - **灵活扩展**：支持自定义解码、加载逻辑。

- **适用场景**：
    - 需要加载网络图片、GIF。
    - 对内存和性能要求较高的场景。

通过理解 Glide 的原理，可以更好地优化图片加载流程，提升应用性能。