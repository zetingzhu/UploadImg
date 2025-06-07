# IntentService 解析和使用  JobIntentService

### Service

不是运行在独立的线程，所以不建议在Service中编写耗时的逻辑和操作，否则会引起ANR。

### IntentService

可用于执行后台耗时的任务，任务执行后会自动停止。

具有高优先级，适合高优先级的后台任务，且不容易被系统杀死。

可以多次启动，每个耗时操作都会以工作队列的方式在IntentService的onHandleIntent（）回调方法中执行。

**源码分析**

```java
// IntentService源码中的 onCreate() 方法
@Override
public void onCreate() {
    super.onCreate();
    // HandlerThread继承自Thread，内部封装了 Looper
    //通过实例化andlerThread新建线程并启动
    //所以使用IntentService时不需要额外新建线程
    HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
    thread.start();

    //获得工作线程的 Looper，并维护自己的工作队列
    mServiceLooper = thread.getLooper();
    //将上述获得Looper与新建的mServiceHandler进行绑定
    //新建的Handler是属于工作线程的。
    mServiceHandler = new ServiceHandler(mServiceLooper);
}

private final class ServiceHandler extends Handler {
public ServiceHandler(Looper looper) {
    super(looper);
}

//IntentService的handleMessage方法把接收的消息交给onHandleIntent()处理
//onHandleIntent()是一个抽象方法，使用时需要重写的方法
@Override
public void handleMessage(Message msg) {
    // onHandleIntent 方法在工作线程中执行，执行完调用 stopSelf() 结束服务。
    onHandleIntent((Intent)msg.obj);
    //onHandleIntent 处理完成后 IntentService会调用 stopSelf() 自动停止。
    stopSelf(msg.arg1);
}

////onHandleIntent()是一个抽象方法，使用时需要重写的方法
@WorkerThread
protected abstract void onHandleIntent(Intent intent);


@Override
public void onStart(@Nullable Intent intent, int startId) {
    Message msg = mServiceHandler.obtainMessage();
    msg.arg1 = startId;
    msg.obj = intent;
    //原来这个Handler在onCreate创建，onStart发送消息
    mServiceHandler.sendMessage(msg);
}
```

### 为什么IntentService具有高优先级

```java
@Override
public void onCreate() {
    // TODO: It would be nice to have an option to hold a partial wakelock
    // during processing, and to have a static startService(Context, Intent)
    // method that would launch the service & hand off a wakelock.

    super.onCreate();
    //新建一个子线程
    HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
    thread.start();
    mServiceLooper = thread.getLooper();
    mServiceHandler = new ServiceHandler(mServiceLooper);
}
```

我们点进去HandlerThread 里面看，发现HandlerThread的构造函数里赋值了一个优先级

```java
public HandlerThread(String name) {
    super(name);
    mPriority = Process.THREAD_PRIORITY_DEFAULT;
}
```

```java
/**
 * Standard priority of application threads.
 * Use with {@link #setThreadPriority(int)} and
 * {@link #setThreadPriority(int, int)}, <b>not</b> with the normal
 * {@link java.lang.Thread} class.
 */
public static final int THREAD_PRIORITY_DEFAULT = 0;
```



使用

创建MyIntentService

```java
public class MyIntentService extends IntentService {

    private static String serviceName = "abc";
    private String TAG = MyIntentService.class.getSimpleName();

    public static final String ACTION_1 = "com.zzt.test.MyIntentService1";
    public static final String ACTION_2 = "com.zzt.test.MyIntentService2";

    public MyIntentService() {
        super(serviceName);
    }

    @Override
    protected void onHandleIntent(@Nullable Intent intent) {
        if (intent != null) {
            String action = intent.getAction();
            for (int i = 0; i < 5; i++) {
                Log.w(TAG, "MyIntentService service:" + serviceName + " ACTION:" + action + " i:" + i);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

配置 AndroidManifest.xml

```java
<service
    android:name=".intentservice.MyIntentService"
    android:exported="false">
    <intent-filter>
        <action android:name="com.zzt.test.MyIntentService1" />
        <action android:name="com.zzt.test.MyIntentService2" />
    </intent-filter>
</service>
```

启动服务

```java
// 创建1
Intent intent1 = new Intent(this, MyIntentService.class);
intent1.setAction(MyIntentService.ACTION_1);
startService(intent1);
// 创建2
Intent intent2 = new Intent(this, MyIntentService.class);
intent2.setAction(MyIntentService.ACTION_2);
startService(intent2);
// 再次启动
startService(intent1);
```

启动结果显示

```java
MyIntentService service:abc ACTION:com.zzt.test.MyIntentService1 i:0   
MyIntentService service:abc ACTION:com.zzt.test.MyIntentService1 i:1   
MyIntentService service:abc ACTION:com.zzt.test.MyIntentService1 i:2   
MyIntentService service:abc ACTION:com.zzt.test.MyIntentService1 i:3   
MyIntentService service:abc ACTION:com.zzt.test.MyIntentService1 i:4   
MyIntentService service:abc ACTION:com.zzt.test.MyIntentService2 i:0   
MyIntentService service:abc ACTION:com.zzt.test.MyIntentService2 i:1   
MyIntentService service:abc ACTION:com.zzt.test.MyIntentService2 i:2   
MyIntentService service:abc ACTION:com.zzt.test.MyIntentService2 i:3   
MyIntentService service:abc ACTION:com.zzt.test.MyIntentService2 i:4   
MyIntentService service:abc ACTION:com.zzt.test.MyIntentService1 i:0   
MyIntentService service:abc ACTION:com.zzt.test.MyIntentService1 i:1   
MyIntentService service:abc ACTION:com.zzt.test.MyIntentService1 i:2   
MyIntentService service:abc ACTION:com.zzt.test.MyIntentService1 i:3   
MyIntentService service:abc ACTION:com.zzt.test.MyIntentService1 i:4   
```





**JobIntentService与IntentService的区别**



```java
public class MyJobIntentService extends JobIntentService {
    private String TAG = MyJobIntentService.class.getSimpleName();
    static final int JOB_ID = 10111;

    @Override
    protected void onHandleWork(@NonNull Intent intent) {
        Log.w(TAG, "MyJobIntentService  启动成功");
        if (intent != null) {
            String jobService = intent.getStringExtra("jobService");
            String action = intent.getAction();
            for (int i = 0; i < 5; i++) {
                Log.w(TAG, "MyJobIntentService  ACTION:" + action + " - jobService:" + jobService + " i:" + i);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    static void enqueueWork(Context context, Intent work) {
        enqueueWork(context, MyJobIntentService.class, JOB_ID, work);
    }

}
```

配置信息

 必须配置这句  android:permission="android.permission.BIND_JOB_SERVICE"

添加权限 <uses-permission android:name="android.permission.WAKE_LOCK" />

```java
<service
    android:name=".intentservice.MyJobIntentService"
    android:exported="false"
    android:permission="android.permission.BIND_JOB_SERVICE">
    <intent-filter>
        <action android:name="com.zzt.test.MyIntentService1" />
        <action android:name="com.zzt.test.MyIntentService2" />
    </intent-filter>
</service>

<!--使用jobIntentService -->
<uses-permission android:name="android.permission.WAKE_LOCK" />
```

使用方法

```java
Intent intent3 = new Intent();
intent3.setAction(MyIntentService.ACTION_1);
intent3.putExtra("jobService", "Job 1");
MyJobIntentService.enqueueWork(this, intent3);
Intent intent4 = new Intent();
intent4.setAction(MyIntentService.ACTION_2);
intent4.putExtra("jobService", "Job 2");
MyJobIntentService.enqueueWork(this, intent4);

intent3.putExtra("jobService", "Job 3");
MyJobIntentService.enqueueWork(this, intent3);
```

打印日志

```java
MyJobIntentService  启动成功
MyJobIntentService  ACTION:com.zzt.test.MyIntentService1 - jobService:Job 1 i:0
MyJobIntentService  ACTION:com.zzt.test.MyIntentService1 - jobService:Job 1 i:1
MyJobIntentService  ACTION:com.zzt.test.MyIntentService1 - jobService:Job 1 i:2
MyJobIntentService  ACTION:com.zzt.test.MyIntentService1 - jobService:Job 1 i:3
MyJobIntentService  ACTION:com.zzt.test.MyIntentService1 - jobService:Job 1 i:4
MyJobIntentService  启动成功
MyJobIntentService  ACTION:com.zzt.test.MyIntentService2 - jobService:Job 2 i:0
MyJobIntentService  ACTION:com.zzt.test.MyIntentService2 - jobService:Job 2 i:1
MyJobIntentService  ACTION:com.zzt.test.MyIntentService2 - jobService:Job 2 i:2
MyJobIntentService  ACTION:com.zzt.test.MyIntentService2 - jobService:Job 2 i:3
MyJobIntentService  ACTION:com.zzt.test.MyIntentService2 - jobService:Job 2 i:4
MyJobIntentService  启动成功
MyJobIntentService  ACTION:com.zzt.test.MyIntentService1 - jobService:Job 3 i:0
MyJobIntentService  ACTION:com.zzt.test.MyIntentService1 - jobService:Job 3 i:1
MyJobIntentService  ACTION:com.zzt.test.MyIntentService1 - jobService:Job 3 i:2
MyJobIntentService  ACTION:com.zzt.test.MyIntentService1 - jobService:Job 3 i:3
MyJobIntentService  ACTION:com.zzt.test.MyIntentService1 - jobService:Job 3 i:4
```

