---
title: "Android 根据自定义View生成图片并保存至相册"
date: 2024-10-17T10:30:23+08:00
tags: ["Android"]
categories: [ "移动端"]
draft: false
---

### 一、保存View的2种方法

- 保存已经显示的View
```kotlin
    /**
     * 注释: 从可见的View中保存Bitmap
     * 时间: 2024/10/17 0017 11:08
     * @param
     * @return
     */
    fun createBitmapFromView(v: View?): Bitmap? {
        if (v == null) {
            return null
        }
        val screenshot: Bitmap =
            Bitmap.createBitmap(v.width, v.height, Bitmap.Config.ARGB_8888)
        val c = Canvas(screenshot)
        c.translate(-v.scrollX.toFloat(), -v.scrollY.toFloat())
        v.draw(c)
        return screenshot
    }
```
- 保存未显示的View
```kotlin
    /**
     * 注释：从不可见View中创建Bitmap
     * 时间：2024/10/17 0017 16:01
     * 作者：郭翰林
     */
    fun createBitmapFromInvisibleView(v: View, dpWidth: Int, dpHeight: Int): Bitmap? {
        //测量使得view指定大小
        val measuredWidth =
            View.MeasureSpec.makeMeasureSpec(
                dpToPixels(v.context, dpWidth),
                View.MeasureSpec.EXACTLY
            )
        val measuredHeight =
            View.MeasureSpec.makeMeasureSpec(
                dpToPixels(v.context, dpHeight),
                View.MeasureSpec.EXACTLY
            )
        v.measure(measuredWidth, measuredHeight)
        //调用layout方法布局后，可以得到view的尺寸大小
        v.layout(0, 0, v.measuredWidth, v.measuredHeight)
        val bmp = Bitmap.createBitmap(v.width, v.height, Bitmap.Config.ARGB_8888)
        val c = Canvas(bmp)
        c.drawColor(Color.WHITE)
        v.draw(c)
        return bmp
    }
```

### 二、保存图片到相册
```kotlin
    /**
     * 注释：保存图片到相册
     * 时间：2024/10/17 0017 15:42
     * 作者：郭翰林
     */
    private fun saveImageToGallery(context: Context, bmp: Bitmap): String? {
        val dataTake = System.currentTimeMillis()
        val jpegName = "IMG_$dataTake.jpg"
        val values = ContentValues()
        values.put(MediaStore.Images.Media.DISPLAY_NAME, jpegName)
        values.put(MediaStore.Images.Media.MIME_TYPE, "image/jpeg")
        values.put(MediaStore.Images.Media.RELATIVE_PATH, "DCIM/Camera")
        val external: Uri
        val resolver = context.contentResolver
        val status = Environment.getExternalStorageState()
        // 判断是否有SD卡,优先使用SD卡存储,当没有SD卡时使用手机存储
        external = if (status == Environment.MEDIA_MOUNTED) {
            MediaStore.Images.Media.EXTERNAL_CONTENT_URI
        } else {
            MediaStore.Images.Media.INTERNAL_CONTENT_URI
        }
        val insertUri = resolver.insert(external, values) ?: return ""
        val os: OutputStream?
        return try {
            os = resolver.openOutputStream(insertUri)
            bmp.compress(Bitmap.CompressFormat.JPEG, 100, os)
            if (os != null) {
                os.flush()
                os.close()
            }
            insertUri.toString()
        } catch (e: IOException) {
            e.printStackTrace()
            ""
        }
    }
```