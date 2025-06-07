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

## 2.引入异步部分

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

# SDK使用部分

## 1.XtrendSdk方法介绍

### startLoop

```java
  /**
     * 设置开始循环
     * @param context
     * @param time  时间单位秒（s）
     */
    public void startLoop(Context context, long time)
```

如果需要修改sdk中的循环时间讲个需要通过startLoop 方法来修改，默认不设置未60秒

### dumpExceptionDb

```
/**
     * 将自己的异常退栈信息写入到数据库中
     */
    public void dumpExceptionDb(Throwable ex) 
```

如果出事设置了不接入自带的异常捕获，需要将自己的异常错误信息手动接入到sdk中

使用方法

```java
public class SDKCrashHandler implements Thread.UncaughtExceptionHandler {
    @Override
    public void uncaughtException(Thread thread, Throwable ex) {
        // 导出异常信息到数据库
        dumpExceptionDb(ex);
    }
}
```

### setStartAppNotice

```java
/**
     * 通知栏启动调用
     */
    public void setStartAppNotice(Context mContext)
```

通过通知栏或者是第三方启动需要自己调用启动方式

### setExitUser

```java
/**
 * 用户退出
 */
public void setExitUser(Context mContext)
```

如果有明确的手动退出应用需要记录

### setClickEvent

```java
/**
 * 用户点击事件
 */
public void setClickEvent(Context context, String event, String value)
public void setClickEvent(Context context, String deviceId, String userId, String event, String value)
```

点击事件，将需要的埋点信息存入到这里

### setAppDeviceID

```java
/**
 * 设置设备id
 */
public void setAppDeviceID(String appDeviceID)
```

在点击事件中没有传入deviceId ，需要在设备启动时候继续自己的设备信息

### setAppUserId

```java
/**
 * 设备用户id
 */
public void setAppUserId(String appUserId) 
```

点击事件中不传入userid 需要在每次登录成功后记录自己的用户信息
