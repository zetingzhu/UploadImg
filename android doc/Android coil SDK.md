Android coil SDK

# 集成

~~~groovy
android {
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = "1.8"
        freeCompilerArgs += "-Xjvm-default=all" // Only required for 2.x.
    }
}
~~~

依赖库

mavenCentral()

```java
//依赖于io.coil-kt:coil-base，提供了 Coil 类的单例对象以及 ImageView 相关的扩展函数
implementation("io.coil-kt:coil:1.4.0")
// 基础组件，提供了基本的图片请求、图片解码、图片缓存、Bitmap 复用等功能
//    implementation("io.coil-kt:coil-base:1.4.0")

//选择添加
implementation("io.coil-kt:coil-gif:1.4.0")//支持GIF
implementation("io.coil-kt:coil-svg:1.4.0")//支持SVG
implementation("io.coil-kt:coil-video:1.4.0")//支持Video

//对 Jetpack Compose 的支持
implementation(" io.coil-kt:coil-compose:1.4.0")
//对 Jetpack Compose 的支持的单例函数 .io.coil-kt:coil-composeImageLoader
implementation("io.coil-kt:coil-compose-base:1.4.0")
```



# 使用

## 快速使用 load

```kotlin
// URL
imageView.load("https://www.example.com/image.jpg")

// Resource
imageView.load(R.drawable.image)

// File
imageView.load(File("/path/to/image.jpg"))

// And more...
```

## lambad 使用

~~~kotlin
imageView.load("https://www.example.com/image.jpg") {
    crossfade(true) //淡入淡出
    placeholder(R.drawable.image) //占位图
    transformations(CircleCropTransformation()) //图片变换，将图片转为圆形
}
~~~

## java中是用使用

~~~java
ImageRequest request = new ImageRequest.Builder(this)
                .data(url)
                .crossfade(true)
                .target(iv)
                .build();
ImageLoader imageLoader = Coil.imageLoader(this);
imageLoader.enqueue(request);

ImageRequest request = new ImageRequest.Builder(context)
    .data("https://www.example.com/image.jpg")
    .size(1080, 1920)
    .build();
Drawable drawable = ImageLoaders.executeBlocking(imageLoader, request).getDrawable();
~~~

## 构建ImageRequest 自定义target 位置

~~~kotlin
val request = ImageRequest.Builder(this)
    .data(ImgUrlConfig.IMG_URL_2)
    .target { drawable ->
			iv_view6?.setImageDrawable(drawable)
            }
	.build()
baseContext.imageLoader.enqueue(request)
~~~

## enqueue：

将要在后台线程上异步执行的 排队。ImageRequest



## execute：

在当前协程中执行 ，并返回ImageResult。ImageRequest

~~~kotlin
 GlobalScope.launch(Dispatchers.Unconfined) {
            val request2 = ImageRequest.Builder(con)
                .data(ImgUrlConfig.IMG_URL_1)
                .build()
            val drawable = imageLoader.execute(request = request2).drawable
            iv.setImageDrawable(drawable)
        }
~~~

## ImageLoader 实例

~~~kotlin
class MyApplication : Application(), ImageLoaderFactory {

    override fun newImageLoader(): ImageLoader {
        return ImageLoader.Builder(applicationContext)
            .crossfade(true)
            .okHttpClient {
                OkHttpClient.Builder()
                    .cache(CoilUtils.createDefaultCache(applicationContext))
                    .build()
            }
            .build()
    }
}
~~~

或者

~~~kotlin
val imageLoader = ImageLoader.Builder(context)
    .crossfade(true)
    .okHttpClient {
        OkHttpClient.Builder()
            .cache(CoilUtils.createDefaultCache(context))
            .build()
    }
    .build()
Coil.setImageLoader(imageLoader)
~~~

获取ImadeLoad 单例

~~~Kotlin
val imageLoader = context.imageLoader
~~~

## 预加载

要将映像预加载到内存中，请排队或执行 不带 ：ImageRequest Target

~~~kotlin
val request = ImageRequest.Builder(context)
    .data("https://www.example.com/image.jpg")
    // Optional, but setting a ViewSizeResolver will conserve memory by limiting the size the image should be preloaded into memory at.
    .size(ViewSizeResolver(imageView))
    .build()
imageLoader.enqueue(request)
~~~



## 取消请求

~~~kotlin
val disposable = imageView.load("https://www.example.com/image.jpg")

//取消正在进行的图片加载请求以及释放相关的资源
disposable.dispose()

//非阻塞式地等待任务结束
disposable.await()
~~~

## Jetpack Compose Coil 插件

[Coil - Jetpack Compose](https://docs.compose.net.cn/third-party-component/coil/)

## 缓存策略有如下几种:

CachePolicy.ENABLED : 可读可写
CachePolicy.READ_ONLY : 只读
CachePolicy.WRITE_ONLY : 只写
CachePolicy.DISABLED : 不可读不可写，即禁用

```kotlin
iv.load(url) {
    crossfade(true) //淡入淡出
    crossfade(1000)//设置显示动画的时间
    memoryCachePolicy(CachePolicy.ENABLED)//设置内存的缓存策略
    diskCachePolicy(CachePolicy.ENABLED)//设置磁盘的缓存策略
    networkCachePolicy(CachePolicy.ENABLED)//设置网络的缓存策略
}


要仅将网络映像预加载到磁盘缓存中，请禁用请求的内存缓存：
val request = ImageRequest.Builder(context)
    .data("https://www.example.com/image.jpg")
    .memoryCachePolicy(CachePolicy.DISABLED)
    .build()
imageLoader.enqueue(request)
```

## Coil默认提供了四种变换:

BlurTransformation() : 高斯模糊变换
CircleCropTransformation() : 圆形裁剪变换
GrayscaleTransformation() : 灰度变换
RoundedCornersTransformation() : 圆角变换

```kotlin
iv.load(url) {
    transformations(
        CircleCropTransformation(),
        RoundedCornersTransformation(
            topLeft = 20f,
            topRight = 40f,
            bottomLeft = 5f,
            bottomRight = 80f
        ),
        BlurTransformation(con, 2f, 2f),
        GrayscaleTransformation(),
        WatermarkTransformation("自定义水印", Color.parseColor("#8D3700B3"), 120f)
    )
}
```

自定义转换类型 WatermarkTransformation

~~~kotlin
package com.zzt.coilsample.util

import android.graphics.Bitmap
import android.graphics.Canvas
import android.graphics.Color
import android.graphics.Paint
import androidx.annotation.ColorInt
import coil.bitmap.BitmapPool
import coil.size.Size
import coil.transform.Transformation

/**
 * @author: zeting
 * @date: 2022/1/25
 * 为图片添加水印
 *
 * //使用自定义的变换
imageView.load(imageUrl) {
transformations(
WatermarkTransformation("自定义", Color.parseColor("#8D3700B3"), 120f)
)
}
 */
class WatermarkTransformation(private val watermark: String, @ColorInt private val textColor: Int, private val textSize: Float) :
    Transformation {

    override fun key(): String {
        return "${WatermarkTransformation::class.java.name}-${watermark}-${textColor}-${textSize}"
    }

    override suspend fun transform(pool: BitmapPool, input: Bitmap, size: Size): Bitmap {
        val width = input.width
        val height = input.height
        val config = input.config

        val output = pool.get(width, height, config)

        val canvas = Canvas(output)
        val paint = Paint()
        paint.isAntiAlias = true
        canvas.drawBitmap(input, 0f, 0f, paint)

        canvas.rotate(40f, width / 2f, height / 2f)

        paint.textSize = textSize
        paint.color = textColor

        val textWidth = paint.measureText(watermark)

        canvas.drawText(watermark, (width - textWidth) / 2f, height / 2f, paint)

        return output
    }

}
~~~
