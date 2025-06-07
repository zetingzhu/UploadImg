Android 线程功能

#  HandlerThread
HandlerThread是一种使用Handler处理消息队列的线程。它继承自Thread类，内部封装了一个Looper，可以处理来自其他线程的消息。

### 使用方法
- 创建一个HandlerThread实例，并调用start()方法启动线程。
- 调用getLooper()方法获取HandlerThread的Looper，并用它创建一个Handler实例。
- 使用Handler的post()、postDelayed()或sendMessage()方法将任务发送到HandlerThread的消息队列。

### 技巧
- 在不再需要HandlerThread时，调用quit()或quitSafely()方法来停止线程。
- 可以使用HandlerThread来处理多个相关任务，以避免频繁地创建和销毁线程。
- 在Android中，IntentService和JobIntentService都是基于Service组件实现的后台服务。它们的工作原理主要通过创建工作线程来在后台执行任务，以避免阻塞主线程。下面我们结合Android源码，来看一下它们是如何实现后台运行的。

### 使用案例（模拟下载）

1. DownloadThread 代码
```java
public class DownloadThread extends HandlerThread{

    private static final String TAG = "DownloadThread";

    public static final int TYPE_START = 2;//通知主线程任务开始
    public static final int TYPE_FINISHED = 3;//通知主线程任务结束

    private Handler mUIHandler;//主线程的Handler

    public DownloadThread(String name) {
        super(name);
    }

    /*
    * 执行初始化任务
    * */
    @Override
    protected void onLooperPrepared() {
        Log.e(TAG, "onLooperPrepared: 1.Download线程开始准备");
        super.onLooperPrepared();
    }

    //注入主线程Handler
    public void setUIHandler(Handler UIhandler) {
        mUIHandler = UIhandler;
        Log.e(TAG, "setUIHandler: 2.主线程的handler传入到Download线程");
    }

    //Download线程开始下载
    public void startDownload() {
        Log.e(TAG, "startDownload: 3.通知主线程,此时Download线程开始下载");
        mUIHandler.sendEmptyMessage(TYPE_START);

        //模拟下载
        Log.e(TAG, "startDownload: 5.Download线程下载中...");
        SystemClock.sleep(2000);

        Log.e(TAG, "startDownload: 6.通知主线程,此时Download线程下载完成");
        mUIHandler.sendEmptyMessage(TYPE_FINISHED);
    }
} 
```

2. UI 使用部分和处理消息。
```java
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    private DownloadThread mHandlerThread;//子线程
    private Handler mUIhandler;//主线程的Handler

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //初始化，参数为线程的名字
        mHandlerThread = new DownloadThread("mHandlerThread");
        //调用start方法启动线程
        mHandlerThread.start();
        //初始化Handler，传递 mHandlerThread 内部的一个 looper
        mUIhandler = new Handler(mHandlerThread.getLooper()) {
            @Override
            public void handleMessage(Message msg) {
                //判断mHandlerThread里传来的msg，根据msg进行主页面的UI更改
                switch (msg.what) {
                    case DownloadThread.TYPE_START:
                        //不是在这里更改UI哦，只是说在这个时间，你可以去做更改UI这件事情，改UI还是得在主线程。
                        Log.e(TAG, "4.主线程知道Download线程开始下载了...这时候可以更改主界面UI");
                        break;
                    case DownloadThread.TYPE_FINISHED:
                        Log.e(TAG, "7.主线程知道Download线程下载完成了...这时候可以更改主界面UI，收工");
                        break;
                    default:
                        break;
                }
                super.handleMessage(msg);
            }
        };
        //子线程注入主线程的mUIhandler，可以在子线程执行任务的时候，随时发送消息回来主线程
        mHandlerThread.setUIHandler(mUIhandler);
        //子线程开始下载
        mHandlerThread.startDownload();
    }

    @Override
    protected void onDestroy() {
        //有2种退出方式
        mHandlerThread.quit();
        //mHandlerThread.quitSafely(); 需要API>=18
        super.onDestroy();
    }
} 
```

### 循环下载使用
```java

/**
 * Created by hj on 2018/12/21.
 * 说明：模拟线程下载
 */
public class DownloadHandlerThread extends HandlerThread implements Handler.Callback {

    private Handler workHandler; //处理下载逻辑的Handler
    private Handler uiHandler; //处理UI刷新的Handler

    public final static int READY = 0X110;
    public final static int STAR = 0X111;
    public final static int STOP = 0X112;
    public final static int ERROR = 0X113;


    private int max;
    private int index;

    public DownloadHandlerThread() {
        super("download-thread");
    }

    @Override
    protected void onLooperPrepared() {
        //初始化
        workHandler = new Handler(getLooper(), this);
        if (uiHandler == null) {
            throw new NullPointerException("uiHandler is not null");
        }
        uiHandler.sendEmptyMessage(READY); //通知主线程Looper已经准备完毕，可以开始下载了
    }

    @Override
    public boolean handleMessage(Message msg) {
        switch (msg.what) {
            case STAR: //开始下载
                index++; //记录当前下载到了第几个
                String url = msg.getData().getString("url");
                Log.i("HJ", "准备下载：" + url);
                try {
                    sleep(2000); //模拟下载
                    if (index == 2) {  //下载到第二个的时候模拟下载失败
                        Log.i("HJ", "下载出错：" + url);
                        setErrorMessage(url);
                        Log.i("HJ", "开始下载下一个地址");
                    } else {
                        Log.i("HJ", "当前地址下载完成：" + url);
                    }
                } catch (InterruptedException e) {
                    setErrorMessage(url);
                    e.printStackTrace();
                }
                if (index == max) { //全部下载完毕
                    workHandler.sendEmptyMessage(STOP);
                }
                break;
            case STOP: //全部下载完成

                //退出Looper循环
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR2) {
                    quitSafely();
                } else {
                    quit();
                }
                Log.i("HJ", "全部下载完成");
                break;
        }
        return false;
    }

    //发送下载错误消息到主线程用于刷新UI
    private void setErrorMessage(String url) {
        Message errorMsg = new Message();
        errorMsg.what = ERROR;
        Bundle bundle = new Bundle();
        bundle.putString("errorUrl", url);
        errorMsg.setData(bundle);
        uiHandler.sendMessage(errorMsg);
    }

    public void setUiHandler(Handler uiHandler) {
        this.uiHandler = uiHandler;
    }

    public Handler getWorkHandler() {
        return workHandler;
    }

    public void setMax(int max) {
        this.max = max;
    }
}
```
使用
```java
public class MainActivity extends AppCompatActivity {

    private Handler uiHandler;
    //模拟数据源
    private String[] urls = new String[]{
            "url-1", "url-2", "url-3", "url-4", "url-5"
    };
    private DownloadHandlerThread mThread;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initUiHandler();
        initDownLoad();
    }

    @SuppressLint("HandlerLeak")
    private void initUiHandler() {
        uiHandler = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                switch (msg.what) {
                    case DownloadHandlerThread.READY: //获取到workHandler已经初始化完毕，可以进行下载任务了
                        Handler workHandler = mThread.getWorkHandler();
                        mThread.setMax(urls.length);
                        for (String url : urls) {  //循环将Message加入MessageQueue消息池中
                            Message message = new Message();
                            message.what = DownloadHandlerThread.STAR;
                            Bundle bundle = new Bundle();
                            bundle.putString("url", url);
                            message.setData(bundle);
                            workHandler.sendMessage(message);
                        }
                        break;
                    case DownloadHandlerThread.ERROR:  //处理下载失败UI逻辑
                        String url = msg.getData().getString("errorUrl");
                        showShortToast("下载失败链接:" + url);
                        Log.i("HJ","展示下载失败UI>>>>>>>>");
                        break;
                }

            }
        };
    }

    private void showShortToast(String string) {
        Toast.makeText(this, string, Toast.LENGTH_SHORT).show();
    }

    private void initDownLoad() {
        mThread = new DownloadHandlerThread();
        mThread.setUiHandler(uiHandler);
        mThread.start();
    }
}
```


### 轮询任务(定时器)
```java
import android.os.Handler;
import android.os.HandlerThread;
import android.os.Message;

public class MyHandlerThread {

    private static final int MSG_POLL = 1;
    private HandlerThread mHandlerThread;
    private Handler mHandler;

    public MyHandlerThread() {
        // 初始化HandlerThread
        mHandlerThread = new HandlerThread("MyHandlerThread");
        mHandlerThread.start();

        // 创建与HandlerThread相关联的Handler
        mHandler = new Handler(mHandlerThread.getLooper(), new Handler.Callback() {
            @Override
            public boolean handleMessage(Message msg) {
                if (msg.what == MSG_POLL) {
                    // 执行你的任务
                    // ...

                    // 重新安排下一个任务
                    sendEmptyMessageDelayed(MSG_POLL, 5000); // 每5秒执行一次
                    return true;
                }
                return false;
            }
        });

        // 发送初始消息启动循环任务
        mHandler.sendEmptyMessage(MSG_POLL);
    }

    public void stop() {
        // 停止循环任务
        mHandler.removeMessages(MSG_POLL);
        mHandlerThread.quit();
    }
}
```
使用
```java
MyHandlerThread handlerThread = new MyHandlerThread();
// ... 当不需要执行循环任务时
handlerThread.stop();
```

# 线程池

### 常见的线程池类型及其用法
1. FixedThreadPool（固定大小线程池）
FixedThreadPool是一个拥有固定数量线程的线程池。这种线程池可以用于执行并发数量有限的任务，例如CPU密集型任务。

用法：
```java
ExecutorService fixedThreadPool = Executors.newFixedThreadPool(4);
fixedThreadPool.execute(new Runnable() {
    @Override
    public void run() {
        // 执行任务
    }
});
```
2. CachedThreadPool（缓存线程池）
CachedThreadPool是一个可以根据需要创建新线程的线程池。这种线程池适用于执行大量短时间任务的场景，例如I/O密集型任务。

用法：
```java
ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
cachedThreadPool.execute(new Runnable() {
    @Override
    public void run() {
        // 执行任务
    }
});
```

3. SingleThreadExecutor（单线程线程池）
SingleThreadExecutor是一个只有一个线程的线程池。这种线程池可以用于保证任务按顺序执行，例如需要顺序处理的任务。

用法：
```java
ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
singleThreadExecutor.execute(new Runnable() {
    @Override
    public void run() {
        // 执行任务
    }
});
```

4. ScheduledThreadPoolExecutor（定时线程池）
ScheduledThreadPoolExecutor是一个可以执行定时任务或周期性任务的线程池。

用法：
```java
ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(4);
// 延迟1秒后执行任务
scheduledThreadPool.schedule(new Runnable() {
    @Override
    public void run() {
        // 执行任务
    }
}, 1, TimeUnit.SECONDS);

// 延迟1秒后，每隔2秒执行一次任务
scheduledThreadPool.scheduleAtFixedRate(new Runnable() {
    @Override
    public void run() {
        // 执行任务
    }
}, 1, 2, TimeUnit.SECONDS);
```

### 技巧
- 根据任务的特点选择合适的线程池类型。例如，对于CPU密集型任务，可以使用固定大小的线程池；对于I/O密集型任务，可以使用缓存线程池。
- 在不再需要线程池时，调用
<font color="red">shutdown()</font>或
<font color="red">shutdownNow()</font>
方法来关闭线程池，释放资源。

### 自定义线程池
ThreadPoolExecutor 是 Java 并发库中的一个类，用于创建和管理自定义线程池。它提供了灵活的线程池配置选项，适用于各种任务场景。以下是关于 ThreadPoolExecutor 的用法的详细阐述。

1. 创建 ThreadPoolExecutor
要创建 ThreadPoolExecutor，请使用其构造函数：
```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
        int corePoolSize, // 核心线程数
        int maximumPoolSize, // 最大线程数
        long keepAliveTime, // 空闲线程的存活时间
        TimeUnit unit, // 存活时间的单位
        // 当线程池达到corePoolSize时，新提交任务将被放入workQueue中，
       // 等待线程池中任务调度执行
        BlockingQueue<Runnable> workQueue, // 工作队列
        // 线程工厂，为线程池提供创建新线程的功能，它是一个接口，
       // 只有一个方法：Thread newThread(Runnable r)
        ThreadFactory threadFactory, // 线程工厂
        // 线程池对拒绝任务的处理策略。一般是队列已满或者无法成功执行任务，
        // 这时ThreadPoolExecutor会调用handler的rejectedExecution
        // 方法来通知调用者
        RejectedExecutionHandler handler // 拒绝策略
);
```
2. 提交任务
使用 execute() 方法提交 Runnable 任务：
```java
executor.execute(new Runnable() {
    @Override
    public void run() {
        // 执行任务
    }
});
```
或者使用 submit() 方法提交 Callable 任务，该方法会返回一个 Future 对象，可以用于获取任务的执行结果：
```java
Future<String> future = executor.submit(new Callable<String>() {
    @Override
    public String call() {
        // 执行任务并返回结果
        return "Hello, ThreadPoolExecutor!";
    }
});
```
3. 关闭线程池
在不再需要线程池时，调用 shutdown() 方法来关闭线程池。这将等待所有任务完成后关闭线程池：
```java
executor.shutdown();
```
如果需要立即关闭线程池，可以调用 shutdownNow() 方法。这将尝试停止所有正在执行的任务，并返回等待执行的任务列表：
```java
List<Runnable> pendingTasks = executor.shutdownNow();
```
4. 获取线程池状态
ThreadPoolExecutor 提供了一些方法来获取线程池的状态，例如：

- getPoolSize()：获取线程池中的当前线程数。
- getActiveCount()：获取线程池中正在执行任务的线程数。
- getCompletedTaskCount()：获取线程池已完成的任务数。
- getTaskCount()：获取线程池已接收的任务总数（包括已完成、正在执行和等待执行的任务）。
示例：
```java
System.out.println("线程池大小: " + executor.getPoolSize());
System.out.println("活跃线程数: " + executor.getActiveCount());
System.out.println("已完成任务数: " + executor.getCompletedTaskCount());
System.out.println("总任务数: " + executor.getTaskCount());
```
