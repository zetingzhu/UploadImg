
研究Retrofit：理解动态代理，注解，反射，学习它所用到的设计模式，达到自己能手写它的核心实现
研究Okhttp:理解它的请求如何共享同一个Socket,内置连接池，连接复用，gzip压缩，响应缓存，自动重试，底层OKIO,所设计到的模式，尤其是拦截器部分的责任链模式
GreenDao,Romm，掌握对象映射，自动生成代码系列，相关 apt,kapt,ksp,spi相关机制等
MMKV,掌握Protobuf, mmap，关联binder跨进程通信 只copy一次原理
SmartRefreshlayout,掌握自定义view,和它的设计思路
Arouter:掌握路由原理，及ASM字节码插桩
Bugly：掌握崩溃，异常捕获，能自定义Thread.UncaughtExcerptionHandler处理异常方案及对traces.txt文件监控方案并分析
Exoplayer掌握音视频的硬解码，缓存处理，能手写它
glide深入理解它的三部分，with部分，注册编码器，管理请求和生命周期监听，load部分，每个请求单独配置option,into部分，启动请求，加载数据，对数据解码，转码，缓存数据，显示数据
RxJava，掌握它背压模式，观察者模式，如何切换线程，能手写它
MPAndroidChart:掌握android 画布绘制原理，能自己绘制各种图表
直播IM的SDK,掌握音频 视频，编码，解码 原理，rtmp,rtsp推拉流等




# AppFrame


android studio mk 不能预览
双击shift按钮，在弹窗中选中Actions，然后在输入框中输入Choose Boot Java Runtime for the IDE，回车后就会弹出一个新的弹窗。

在这个弹窗中点击下拉列表，选中一个和你的Android studio匹配的版本下载下来然后安装，重启AS就可以了。

https://github.com/zetingzhu/AppFrame

```
include ':mypagingsamples'
paging 分页

groupedadapter
自定义布局分组列表
headerrecycleview
头部分组悬停recycleview
SideHeadRecycleSamples
侧边字母工具栏

MoveRecyclerView
手指移动切换位置

1. 嘿嘿
   把自己平时使用的功能整理起来

DataBindSample
DataBind 简单使用

RoomSample
room 数据库简单使用

```

# AppFrame2
https://github.com/zetingzhu/AppFrame2
```
github taken
3362958fa7dcb6c6b4d9c6e38c920ec1cfb7e359

WorkManagerSample WorkManager 简单使用

NettySample Netty 使用

DataBindSample DataBind 使用

LurCacheSample LruCache 和 disklrucache 使用

coordinatorlayoutsample coordinatorlayout 和 RecycleView onDrawOver 使用

dragviewsample 应用内的悬浮球

MotionLayoutSample MotionLayout 结合 MotionScene 实现页面内的动画，位移旋转，渐变，图片替换 这个功能有点傻逼了，作用很小不在研究

MyZxingSample 二维码使用 Zxinglibrary 二维码library zxingSample 二维码demo

ZZTBanner Banner 循环滚动，支持上下，左右，自定义指示器

drawerlayoutsample drawerlayout 使用

messagev1 AIDL Binder 服务端 messagev2 AIDL Binder 客户端

SideHeadRecycleSamples 侧边字母工具栏
```


# AppFrame3

https://github.com/zetingzhu/AppFrame3
```
startactivity
应用启动页优化，背景设置

simpleapk 手动打包一个apk,了解apk打包流程

contactutilssemple 获取手机通讯录信息

floatlayoutsample 浮动布局，可以根据添加布局款数自动匹配长度换行显示

EmojiCompatSample Emoji 使用

CanvasConfettiSample 五彩纸屑，炸出纸屑烟花

myviewpager ViewPager2 使用

SignatureView 手势签名

viewautosizelayout 屏幕适配
```

# AppFrame4
https://github.com/zetingzhu/AppFrame4
```
EasyCalendar 日历的库 samplecalendar 日历的使用

SampleNestedScrolling 各种 NestedScrolling 列表头部联动应用

MainApp PlugInApp pluginframework 插件化多应用

coroutinessample 携程使用

systemcalendatsample 系统日历使用

StaggeredGridSample RecycleView瀑布流布局使用

zztpopupwindows PopupWindows 使用

samplelivedatamut livedata 粘连性，导致数据多次接收

samplewebview , CacheWebView WebView 缓存
```

# AppFrame5
https://github.com/zetingzhu/AppFrame5
```
zztpopupwindows PopupWindows 工具类

samplecanvas 测试各种画图

SampleDayNight 暗黑模式

SampleShedowLayout 渐变背景

samplerotatetextview 自定义 RotateTextView 带有斜角的文字

sampleleakcanary LeakCanary 内存分析工具和 Jetpack Startup 启动管理

SampleNavigation Navigation 跳转使用

SampleFloatingActionButton SampleFloatingAction 使用记录 FloatingActionButtonLibrary 自定义的底部按钮绑定recycleview 联动组件
```

# AppFrame6
https://github.com/zetingzhu/AppFrame6
~~~
app 这个用来学习apt

SampleWebCookie 获取html cookies

SampleRegisterForActivityResult 学习 activity fragment 的 registerForActivityResult() API

SampleMergeAdapter ConcatAdapter 使用 DiffUtil 使用 ConcatAdapter 结合 DiffUtil 刷新有bug ,数据会错乱

ZZTLoggingInterceptorLog okhttp3 日志

SampleBannerViewPager2 ViewPager2 实现的循环滑动播放广告，带有自定义指示器

HoriSlideSample 横向滑动按钮

zzt-dialogutils 弹框整理
~~~

# AppFrame7
https://github.com/zetingzhu/AppFrame7
```
app

Snackbar 使用

zt-threeprogress 自定义三段是进度条

zt_fulldialog 修改定义全屏对话框，上面一层隐形顶部也能去掉 MyTextInputLayout 自定义的Inputlayout 适配自己程序的输入框，带删除 ，带眼睛，带错误信息展示

Zt-MarqueeText textview 多种跑马灯效果 ， MarqueeForeverTextView修改成自己需要的跑马灯效果

zt-magnifier 放大镜
```

# AppFrame8
https://github.com/zetingzhu/AppFrame8
```
app

zt-trasact
透明 Activity 主题使用

zt-scaleparentviewgroup 自动缩放子布局，用作等比缩放

zt-autocompletetextview AutoCompleteTextView 简单使用

zt-hilt hilt 简单了解

zt-glideapputil Glide 使用，已经简单工具

zt-gson gson 序列号和反序列号解析空处理

zt-cropimageview 裁剪合适图片

zt-lifecycle 声明周期感知简单用法

zt-colorpicker 颜色选择器

zt-cropimageview 图片裁剪，合成

zt-edittextinput 数字输入各种控制
```

# AppFrame9
https://github.com/zetingzhu/AppFrame9
```
zt-fragmentscreenact Fragment 切换

zt-apprightcount 右上角角标

zt-vp2-rv 去 appFrame3->myviewpager 处理一些 viewpager2 和 recycleView 滑动灵敏事件

zt-websocket websocket 各种使用

zt-rv-hsc 列表上下左右滑动

zt-queueList 固定队列，添加一个元素，移除一个最早元素

zt-permiss  android 和权限相关的申请操作
```

# ZZTUtilCode
https://github.com/zetingzhu/ZZTUtilCode
>  android 工具类代码整理

## ByteUtils
byte hex int String 数据处理 --> [ByteUtils][ByteUtils.java]
```
```

## ViewToBitmapUtil
将view 根据内存大小来转换成bitmap --> [ViewToBitmapUtil][viewtobitmaputil.java]
```
使用方法
    // 将view转换成图片，保存图片到本地
    iv_save.setOnClickListener {
        // 初始化view
        val snapshot = ViewToBitmapUtil(ll_save_view)
        // 将view 转换成 bitmap
        val bitmapSrc  = snapshot.apply()

        //系统相册目录
        val path = File.separator + Environment.DIRECTORY_DCIM + File.separator + "Camera" + File.separator
        var appCacheDir: File? = File(Environment.getExternalStorageDirectory(), path)
        // 保存图片完全路径
        val takePhotoFile = File(appCacheDir, "share_" + System.currentTimeMillis() + ".png")
        LogUtils.dTag(TAG, "保存图片路径： " +  takePhotoFile.absolutePath )
        // 图片保存
        val save = ImageUtils.save( bitmapSrc , takePhotoFile , Bitmap.CompressFormat.JPEG)
        LogUtils.dTag(TAG, "保存图片成功状态： " +  save )
        ToastUtils.showShort("保存图片状态：" + save )
        // 最后通知图库更新
        this@ActivityViewToBitmap.getApplicationContext()
            .sendBroadcast(Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE, Uri.fromFile(takePhotoFile)))

    }

```

## ScheduledExecutorManager
线程池定时器 --> [ScheduledExecutorManager][ScheduledExecutorManager.java]
```
使用方法

    mScheduledExecutorManager = ScheduledExecutorManager()
    mScheduledExecutorManager?.scheduleAtFixedRate({
        var sdf = SimpleDateFormat("yyyy-MM-dd HH:mm:ss:SSS", Locale.getDefault() )
        Log.d(TAG , "执行线程时间 1 :" + sdf.format( System.currentTimeMillis() ) )
        TimeUnit.SECONDS.sleep(5 )
    }, 1 , 4 , TimeUnit.SECONDS )

```

# ZT-Canvase
https://github.com/zetingzhu/ZT-Canvase
> 画图相关


# ZZT-RecycleView
https://github.com/zetingzhu/ZZT-RecycleView
> 一下头部滑动相关的 recycleView


# ZT-ComposeSample
https://github.com/zetingzhu/ZT-ComposeSample

# GoogleAutoTest
https://github.com/zetingzhu/GoogleAutoTest
> google 自动登录

# ViewPager2
https://github.com/zetingzhu/zt-ViewPager2
> viewPager2 的使用，替换 fragment 添加一些切换动画


# ZT-ndkSample
ndk 简单实用介绍工程
