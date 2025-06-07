**SDK接入注意事项**

# sdk 引入部分

## 1.引入主线程部分

这一部分主要是接入ApplicationLifecycleObserver和ActivityLifecycleCallbacks 分别来监听应用启动和每个Activity 生命周期

在自定义的Application 中的onCreate() 方法中引入initMainLooper()

```java
/**
     * 生命感知放在主线程
     */
public void initMainLooper(Application context)

XtrendSdk.getInstance().initMainLooper(this);
```

2.引入异步部分

这一部分主要功能是 网络初始化，数据库初始化，崩溃日志初始，定时线程初始化

在自定义的Application 中的onCreate() 方法中引入init(),这一部分可以 异步线程启动，提高应用启动效率

```java
/**
 * Application 中应用初始化
 *
 * @param context
 * @param market    渠道信息
 * @param initCrash 自己有已经监听类不需要初始化默认的异常监听，
 *                  但是需要将异常数据写入导入到自己的异常收集工具中，
 *                  用来保存数据  dumpExceptionDb
 */
public void init(Application context, String market, boolean initCrash)

     new Thread(new Runnable() {
        @Override
        public void run() {
            XtrendSdk.getInstance().init(application, "渠道信息", true);
        }
    }).start();
```



## 埋点其他应用

DoraemonKit 等待转移 

