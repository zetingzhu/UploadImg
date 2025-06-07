**WordManger 使用操作**

***

# WorkManager 集成
```
dependencies {
    def work_version = "2.8.0"
    // (Java only)
    implementation "androidx.work:work-runtime:$work_version"

    // Kotlin + coroutines
    implementation "androidx.work:work-runtime-ktx:$work_version"
}
```

# WorkManager 三种工作类型

类型|周期|使用方式|描述
--|--|--|--
立即|一次性|OneTimeWorkRequest 和 Worker。如需处理加急工作，请对 OneTimeWorkRequest 调用 setExpedited()。|必须立即开始且很快就完成的任务，可以加急。
长期运行|一次性或定期|任意 WorkRequest 或 Worker。在工作器中调用 setForeground() 来处理通知。|运行时间可能较长（有可能超过 10 分钟）的任务。
可延期|一次性或定期|PeriodicWorkRequest 和 Worker。|延期开始并且可以定期运行的预定任务。

> <font color=red> 问题：</font>
>
> <font color=red> 1. 长期运行只实现了一次性前台服务，循环任务不能设置前台</font>
>
> <font color=red>2. 杀掉进程后模拟器可以继续运行，国产手机无法继续运行</font>


***

## 一次性任务
```
val request = OneTimeWorkRequestBuilder<BWork>()
    .setExpedited(OutOfQuotaPolicy.RUN_AS_NON_EXPEDITED_WORK_REQUEST)
    .addTag(TimeUtils.getNowString())
    .build()
WorkManager.getInstance(baseContext)
    .enqueue(request)

class BWork : Worker {
    val TAG = BWork::class.java.simpleName
    var NOTIFICATION_ID = 9999

    constructor(context: Context, workerParams: WorkerParameters) : super(context, workerParams) {
    }

    override fun getForegroundInfo(): ForegroundInfo {
        val notification = NotificationUtils.getNotification(
            ChannelConfig.DEFAULT_CHANNEL_CONFIG
        ) { param ->
            param.setSmallIcon(R.mipmap.ic_launcher)
                .setContentTitle("title")
                .setContentText("content text: $NOTIFICATION_ID")
                .setAutoCancel(true)
        }
        return ForegroundInfo(NOTIFICATION_ID, notification)
    }

    override fun doWork(): Result {
        // 工作

        // 返回工作结果
        return Result.success()
    }
}

```
- 调度加急工作
> 1. 设置 setExpedited()  OutOfQuotaPolicy.RUN_AS_NON_EXPEDITED_WORK_REQUEST
> 2. 实现 getForegroundInfo() 方法
> 3. 限制条件： 加急也是理想情况，如果当前有正在执行任务，线程池是满的，也是需要依次执行

- 失败异常
> <font color=red>
java.lang.IllegalStateException: Expedited WorkRequests require a Worker to provide an implementation for getForegroundInfo()
</font>





## 循环任务
- 固定周期定期任务
> <font color=red>可以定义的最短重复间隔是 15 分钟</font>

```
val cWork1 = PeriodicWorkRequestBuilder<CWork>(15, TimeUnit.MINUTES)
    .addTag("定期>固定周期：" + TimeUtils.getNowString())
    .build()
WorkManager.getInstance(baseContext).enqueue(cWork1)
```

- 固定周期加灵活间隔任务
> 灵活时间段从 （重复间隔 - 灵活间隔） 开始，一直到间隔结束。
>
> eg: 重复间隔15分钟，灵活间隔5分钟，定时任务开始时间是（15-5）=10分钟，每次灵活间隔开始后的10分钟后开始执行
>
> <font color=red>重复间隔必须大于等于15分钟，而灵活间隔必须大于等于 5分钟，小于等于间隔时间。</font>

~~~
val cWork2 = PeriodicWorkRequestBuilder<CWork>(
    15, TimeUnit.MINUTES, // 重复间隔
    5, TimeUnit.MINUTES// 而灵活间
)
    .addTag("定期+灵活周期：" + TimeUtils.getNowString())
    .build()
WorkManager.getInstance(baseContext).enqueue(cWork2)
~~~


## 延期任务
给 WrokRequest 设置 setInitialDelay
```
val aWork3: WorkRequest = OneTimeWorkRequestBuilder<AWork>()
    .addTag("延迟一次性工作：" + TimeUtils.getNowString())
    .setInitialDelay(1, TimeUnit.MINUTES)
    .build()
WorkManager.getInstance(this@WorkActivity).enqueue(aWork3)
```

> PeriodicWorkRequest 也可以设置初始延迟时间。在这种情况下，定期工作只有首次运行时会延迟。

***


# 工作约束

```
val mBuilder = Constraints.Builder()
mBuilder.setRequiredNetworkType(NetworkType.UNMETERED)
mBuilder.setRequiresBatteryNotLow(true)// 电量不能太低
mBuilder.setRequiresStorageNotLow(true)//  存储不能太少
mBuilder.setRequiresCharging(true)// 充电
mBuilder.setRequiresDeviceIdle(true)// 设置空闲
val constraints = mBuilder.build()
val aWork4: WorkRequest = OneTimeWorkRequestBuilder<AWork>()
   .addTag("约束一次性工作：" + TimeUtils.getNowString())
   .setConstraints(constraints)
   .build()
WorkManager.getInstance(this@WorkActivity).enqueue(aWork4)
```
约束可确保将工作延迟到满足最佳条件时运行。以下约束适用于 WorkManager。

|||
|--|--|
|NetworkType|约束运行工作所需的网络类型。例如 Wi-Fi (UNMETERED)。|
|BatteryNotLow|如果设置为 true，那么当设备处于“电量不足模式”时，工作不会运行。
|RequiresCharging|如果设置为 true，那么工作只能在设备充电时运行。
|DeviceIdle|如果设置为 true，则要求用户的设备必须处于空闲状态，才能运行工作。在运行批量操作时，此约束会非常有用；若是不用此约束，批量操作可能会降低用户设备上正在积极运行的其他应用的性能。
|StorageNotLow|如果设置为 true，那么当用户设备上的存储空间不足时，工作不会运行。

1. setRequiredNetworkType

设置需要的网络类型。一共有如下几种。

枚举值|状态说明
----|-----
NOT_REQUIRED|不需要网络
CONNECTED|任何可用网络
UNMETERED|需要不计量网络，如WiFi
NOT_ROAMING|需要非漫游网络
METERED|需要计量网络，如4G

2. setRequiresBatteryNotLow

设置手机电量的状态，这里的条件是执行任务时手机电量不能较低。

3. setRequiresStorageNotLow

设置手机内存空间的状态，这里的条件是执行任务时手机内存空间不能较低。

4. setRequiresCharging

设置手机的充电状态， 这里的条件是手机执行任务时手机是处于充电的状态。

5. setRequiresDeviceIdle

设置手机的状态， 这里的条件是手机执行任务时手机是出于空闲状态。

6. addContentUriTrigger(uri:Uri, triggerForDescendants:Boolean)

添加触发器，即当指定的URI中有内容更新时会触发任务。值得注意的是这里只能在OneTimeWorkRequest中设置。
<font color=red>还没搞懂怎么用的，</font>

# 工作监听
拿到 LiveData<WorkInfo> 设置 observe
- public abstract @NonNull LiveData<WorkInfo> getWorkInfoByIdLiveData(@NonNull UUID id);
-   public abstract @NonNull LiveData<List<WorkInfo>> getWorkInfosByTagLiveData(@NonNull String tag);


State状态|描述
---|----
ENQUEUED|加入队列
RUNNING|运行中
SUCCEEDED|已成功
FAILED|失败
BLOCKED|挂起
CANCELLED|取消

```
val mBuilder = Constraints.Builder()
mBuilder.setRequiredNetworkType(NetworkType.UNMETERED)
val constraints = mBuilder.build()
val eWork: WorkRequest = OneTimeWorkRequestBuilder<EWork>()
    .addTag("工作监听：" + TimeUtils.getNowString())
    .setConstraints(constraints)
    .build()
WorkManager.getInstance(this@WorkActivity).enqueue(eWork)
WorkManager.getInstance(this@WorkActivity)
    .getWorkInfoByIdLiveData(eWork.id)
    .observe(this, object : Observer<WorkInfo> {
        override fun onChanged(t: WorkInfo?) {
            Log.w(TAG, "工作监听 t:" + t.toString())
        }
    })
```
> WorkInfo{mId='b08e08c6-f7f0-43e2-b0e2-a6d9b8694f92', mState=RUNNING, mOutputData=Data {}, mTags=[com.example.zzt.zworkmanager.work.EWork, 工作监听：2023-09-01 10:24:36], mProgress=Data {}}

workinfo 信息
```
private @NonNull UUID mId;
private @NonNull State mState;
private @NonNull Data mOutputData;
private @NonNull Set<String> mTags;
private @NonNull Data mProgress;
```

# Result 返回状态
从 doWork() 返回的 Result 会通知 WorkManager 服务工作是否成功，以及工作失败时是否应重试工作。
- Result.success()：工作成功完成。
- Result.failure()：工作失败。
- Result.retry()：工作失败，应根据其重试政策在其他时间尝试。

```
override fun doWork(): Result {

    ......

    return Result.success()
}
```

# 回退，重试策略
同时设置了 setBackoffCriteria 和 返回doWork() Result.retry() 才会触发回退策略
- backoffPolicy - 设置策略模式
- backoffDelay - 设置第一次重试时长
- timeUnit - 设置时间单位
fun setBackoffCriteria(
    backoffPolicy: BackoffPolicy,
    backoffDelay: Long,
    timeUnit: TimeUnit
)

策略模式|描述
----|-----
EXPONENTIAL|倍数增加 eg:Result.retry() 结束并在 10 秒后重试,接下来依次 20、40、80 秒
LINEAR|线性增加 eg:Result.retry() 结束并在 10 秒后重试,接下来依次 20 、30 、40 秒

```
val mWork: WorkRequest = OneTimeWorkRequestBuilder<FWork>()
    .addTag("重试工作 linear：" + TimeUtils.getNowString())
    .setBackoffCriteria(
        BackoffPolicy.LINEAR,
        WorkRequest.MIN_BACKOFF_MILLIS,
        TimeUnit.MILLISECONDS
    )
    .build()
WorkManager.getInstance(this@WorkActivity).enqueue(mWork)
```

# 入参
fun setInputData(inputData: Data)
```
val output2: Data = workDataOf(
    "param" to "111",
    "aaa" to 123
)
val mWork: WorkRequest = OneTimeWorkRequestBuilder<GWork>()
    .addTag("入参：" + TimeUtils.getNowString())
    .setInputData(output2)
    .build()
WorkManager.getInstance(this@WorkActivity).enqueue(mWork)
```
取值 Worker.getInputData()
```
val param = inputData.getString("param")
```

# 唯一任务
防止同一工作任务充值添加，主要解决重复调度问题
- WorkManager.enqueueUniqueWork()（用于一次性工作）
```
val mWork = OneTimeWorkRequestBuilder<AWork>()
     .addTag("唯一工作，一次性")
     .build()
 WorkManager.getInstance(baseContext).enqueueUniqueWork(
     "aaaaa",
     ExistingWorkPolicy.REPLACE,
     mWork
 )
```
- WorkManager.enqueueUniquePeriodicWork()（用于定期工作）
```
val mWork = PeriodicWorkRequestBuilder<CWork>(15, TimeUnit.MINUTES)
    .addTag("唯一工作,循环：" + TimeUtils.getNowString())
    .setConstraints(
        Constraints.Builder()
            .setRequiresCharging(true)
            .build()
    )
    .build()
WorkManager.getInstance(baseContext).enqueueUniquePeriodicWork(
    "bbbbb",
    ExistingPeriodicWorkPolicy.UPDATE,
    mWork
)
```

这两种方法都接受 3 个参数：
- uniqueWorkName - 用于唯一标识工作请求的 String。
- existingWorkPolicy - 此 enum 可告知 WorkManager：如果已有使用该名称且尚未完成的唯一工作链，应执行什么操作。如需了解详情，请参阅冲突解决政策。
- work - 要调度的 WorkRequest。

## 冲突解决政策

一次性工作4个冲突策略

一次性工作|ExistingWorkPolicy
---|---
REPLACE|用新工作替换现有工作。此选项将取消现有工作。
KEEP|保留现有工作，并忽略新工作。
APPEND|将新工作附加到现有工作的末尾。此政策将导致您的新工作链接到现有工作，在现有工作完成后运行。现有工作将成为新工作的先决条件。如果现有工作变为 CANCELLED 或 FAILED 状态，新工作也会变为 CANCELLED 或 FAILED。如果您希望无论现有工作的状态如何都运行新工作，请改用 APPEND_OR_REPLACE。
APPEND_OR_REPLACE|函数类似于 APPEND，不过它并不依赖于先决条件工作状态。即使现有工作变为 CANCELLED 或 FAILED 状态，新工作仍会运行。

定期工作3个冲突策略

定期工作|ExistingPeriodicWorkPolicy
---|---
UPDATE|等待显有工作执行完成，下次更新用新的工作替换
KEEP|保留现有工作，并忽略新工作。
CANCEL_AND_REENQUEUE|取消显有工作，排队新的工作

# 工作查询
WorkQuery 支持按工作的标记、状态和唯一工作名称的组合进行查询。
- public static Builder fromIds(@NonNull List<UUID> ids)
- public static Builder fromTags(@NonNull List<String> tags)
- public static Builder fromStates(@NonNull List<WorkInfo.State> states)
- public static Builder fromUniqueWorkNames(@NonNull List<String> uniqueWorkNames)

```
val workQuery = WorkQuery.Builder
    /******************************/
    .fromTags(listOf("唯一工作，一次性"))
    .addStates(
        listOf(
            WorkInfo.State.RUNNING,
            WorkInfo.State.ENQUEUED,
            WorkInfo.State.SUCCEEDED,
            WorkInfo.State.FAILED,
            WorkInfo.State.CANCELLED,
            WorkInfo.State.BLOCKED,
        )
    )
    /******************************/
//                        .fromStates(listOf(WorkInfo.State.FAILED, WorkInfo.State.CANCELLED))
//                        .fromStates(
//                            listOf(
//                                WorkInfo.State.RUNNING,
//                                WorkInfo.State.ENQUEUED,
//                                WorkInfo.State.SUCCEEDED,
//                                WorkInfo.State.FAILED,
//                                WorkInfo.State.CANCELLED,
//                                WorkInfo.State.BLOCKED,
//                            )
//                        )
    /******************************/
//                        .addUniqueWorkNames(
//                            listOf("aaaaa", "bbbbb")
//                        )
    .build()
val workInfos = workManager?.getWorkInfosLiveData(workQuery)
workInfos?.observe(this, object : Observer<List<WorkInfo>> {
    override fun onChanged(t: List<WorkInfo>?) {
        Log.w(TAG, "工作监听 21:" + t.toString())
    }
})
```

# 取消结束任务

cancelAllWork(): 取消所有的任务

cancelAllWorkByTag(tag:String): 取消一组带有相同标签的任务

cancelUniqueWoork(uniqueWorkName:String)： 取消唯一任务
```
 workManager?.cancelAllWorkByTag("唯一工作，一次性")
```

# 工作进度设置
setProgressAsync(@NonNull Data data)
可以异步设置进度情况

通过监听 WorkInfo 详情可以知道当前设置进度

```
class H1Work : Worker {
    val TAG = H1Work::class.java.simpleName
    var maxCount = 20

    constructor(context: Context, workerParams: WorkerParameters) : super(context, workerParams) {
        Log.v(TAG, "work 工作进度 创建 tags:$tags")
    }

    override fun doWork(): Result {
        Log.v(
            TAG,
            "work 工作进度 开始工作 tags:$tags  maxCount${maxCount} "
        )
        // 返回工作结果
        return timeWork()
    }

    fun timeWork(): Result {
        var count = 0;
        setProgressAsync(workDataOf("pro" to "0%"))
        while (true) {
            count++
            if (count >= maxCount) {
                break
            }
            setProgressAsync(workDataOf("pro" to "${count}%"))
            Log.v(
                TAG, "work 工作进度 :" + count +
                        "\n  maxCount:${maxCount} tags:$tags " +
                        "ThreadPool:${WorkManager.getInstance(MyApplication.instance.applicationContext).configuration.executor}"
            )
            try {
                TimeUnit.SECONDS.sleep(1)
            } catch (e: Exception) {
                e.printStackTrace()
            }
        }

        setProgressAsync(workDataOf("pro" to "100%"))
        try {
            TimeUnit.SECONDS.sleep(1)
        } catch (e: Exception) {
            e.printStackTrace()
        }
        return Result.success()
    }
}
```

# 工作链接
如需创建工作链，
- 您可以使用 WorkManager.beginWith(OneTimeWorkRequest) 或 WorkManager.beginWith(List<OneTimeWorkRequest>)，这会返回 WorkContinuation 实例。

- 然后，可以使用 WorkContinuation 通过 then(OneTimeWorkRequest) 或 then(List<OneTimeWorkRequest>) 添加 OneTimeWorkRequest 依赖实例。 .

- 最后，您可以使用 WorkContinuation.enqueue() 方法对 WorkContinuation 工作链执行 enqueue() 操作。

```
val mWork1 = OneTimeWorkRequestBuilder<H1Work>().addTag("链接1").addTag("链接任务").build()
val mWork2 = OneTimeWorkRequestBuilder<H1Work>().addTag("链接2").addTag("链接任务").build()
val mWork3 = OneTimeWorkRequestBuilder<H1Work>().addTag("链接3").addTag("链接任务").build()
val mWork4 = OneTimeWorkRequestBuilder<H1Work>().addTag("链接4").addTag("链接任务").build()
val mWork5 = OneTimeWorkRequestBuilder<H1Work>().addTag("链接5").addTag("链接任务").build()
workManager
    ?.beginWith(listOf(mWork1, mWork5, mWork3))
    ?.then(mWork4)
    ?.then(mWork2)
    ?.enqueue()
```
## 同时执行
WorkManager.beginWith(List<OneTimeWorkRequest>) 可以同时执行多个工作
```
workManager
    ?.beginWith(listOf(mWork1, mWork2, mWork3))
    ?.enqueue()
```
## 顺序执行
```
workManager
    ?.beginWith(mWork1)
    ?.then(mWork2)
    ?.then(mWork3)
    ?.enqueue()
```
## 输入结果合并
WorkManager 提供两种不同类型的 InputMerger：

-|-
-|-
OverwritingInputMerger|会尝试将所有输入中的所有键添加到输出中。如果发生冲突，它会覆盖先前设置的键。
ArrayCreatingInputMerger|会尝试合并输入，并在必要时创建数组。

```
class IWork : Worker {
    val TAG = IWork::class.java.simpleName
    var maxCount = 5
    var lastResult: String? = null
    var lastResultArray: Array<String>? = null

    constructor(context: Context, workerParams: WorkerParameters) : super(context, workerParams) {
        Log.v(TAG, "work 工作进度 创建 tags:$tags")
    }

    override fun doWork(): Result {
        lastResult = inputData.getString("result")
        lastResultArray = inputData.getStringArray("result")
        val randoms = (0..10).random()
        maxCount += randoms
        Log.w(
            TAG,
            "work 工作进度 开始工作 tags:$tags  maxCount${maxCount} lastResult:${lastResult} lastResultArray:${
                Arrays.toString(
                    lastResultArray
                )
            }"
        )
        // 返回工作结果
        return timeWork()
    }


    fun timeWork(): Result {
        var count = 0;
        while (true) {
            count++
            if (count >= maxCount) {
                break
            }
            Log.v(
                TAG, "work 工作进度 :" + count +
                        "\n  maxCount:${maxCount} tags:$tags " +
                        "ThreadPool:${WorkManager.getInstance(MyApplication.instance.applicationContext).configuration.executor}"
            )
            try {
                TimeUnit.SECONDS.sleep(1)
            } catch (e: Exception) {
                e.printStackTrace()
            }
        }
        var resultStr = tags.toString()
        if (lastResult?.isNotEmpty() == true) {
            resultStr = resultStr.plus(" > ls:${lastResult} ")
        }
        if (lastResultArray?.isNotEmpty() == true) {
            resultStr = resultStr.plus(" 》》 lsa:${Arrays.toString(lastResultArray)} ")
        }

        val resultData = Data.Builder().putString("result", resultStr).build()
        Log.e(TAG, "work resultData:$resultData")
        return Result.success(resultData)
    }
}
```
- OverwritingInputMerger 覆盖模式输出日志

eg: work3 收到inputData 是 work1 和 work4 setInputData 覆盖后结果，最后设置将会被传递
> work 工作进度 开始工作 tags:[链接结果合并, 链接3, com.example.zzt.zworkmanager.work.IWork]  maxCount15
lastResult:[链接1, 链接结果合并, com.example.zzt.zworkmanager.work.IWork] lastResultArray:null

```
val mWork1 =
    OneTimeWorkRequestBuilder<IWork>().addTag("链接1").addTag("链接结果合并")
        .setInputMerger(OverwritingInputMerger::class.java)
        .build()
val mWork3 =
    OneTimeWorkRequestBuilder<IWork>().addTag("链接3").addTag("链接结果合并")
        .setInputMerger(OverwritingInputMerger::class.java)
        .build()
val mWork4 =
    OneTimeWorkRequestBuilder<IWork>().addTag("链接4").addTag("链接结果合并")
        .setInputMerger(OverwritingInputMerger::class.java)
        .build()
workManager
    ?.beginWith(listOf(mWork1, mWork4))
    ?.then(mWork3)
    ?.enqueue()
```

- ArrayCreatingInputMerger 合并模式输出日志
eg: work3 收到inputData 是 work1 和 work4 setInputData 合并后结果，会组成一个 list 输出
> work 工作进度 开始工作 tags:[链接结果合并, 链接3, com.example.zzt.zworkmanager.work.IWork]  maxCount8
lastResult:null
lastResultArray:[[链接1, 链接结果合并, com.example.zzt.zworkmanager.work.IWork], [链接结果合并, 链接4, com.example.zzt.zworkmanager.work.IWork]]

```
val mWork1 =
    OneTimeWorkRequestBuilder<IWork>().addTag("链接1").addTag("链接结果合并")
        .setInputMerger(ArrayCreatingInputMerger::class.java)
        .build()
val mWork3 =
    OneTimeWorkRequestBuilder<IWork>().addTag("链接3").addTag("链接结果合并")
        .setInputMerger(ArrayCreatingInputMerger::class.java)
        .build()
val mWork4 =
    OneTimeWorkRequestBuilder<IWork>().addTag("链接4").addTag("链接结果合并")
        .setInputMerger(ArrayCreatingInputMerger::class.java)
        .build()
workManager
    ?.beginWith(listOf(mWork1, mWork4))
    ?.then(mWork3)
    ?.enqueue()
```

## 工作链合并
WorkContinuation.combine 可以将两个工作链合并在一起

```
// 组合执行
val chain1: WorkContinuation? = workManager?.beginWith(mWork1)?.then(mWork2)
val chain2: WorkContinuation? = workManager?.beginWith(mWork3)?.then(mWork4)
WorkContinuation
    .combine(listOf(chain1, chain2))
    .then(mWork5)
    .enqueue()
```

## 唯一工作链
通过 WorkManager.beginUniqueWork() 创建 WorkContinuation 实例

```
workManager?.beginUniqueWork(
    "唯一工作链",
    ExistingWorkPolicy.KEEP,
    listOf(mWork1, mWork4)
)
    ?.then(mWork3)
    ?.then(mWork2)
    ?.then(mWork5)
    ?.enqueue()
```


***
# 源码解读



***

# 自定义 WorkManager 配置和初始化

> eg: 自定义线程池，系统默认线程池4个，可以同时启动4个任务


从 WorkManager 2.6 开始，应用启动功能便已在 WorkManager 内部使用。如需提供自定义初始化程序，您需要移除 androidx.startup 节点。

1. 如果您不在应用中使用应用启动功能，则可以将其彻底移除。
```
 <!-- If you want to disable android.startup completely. -->
 <provider
    android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    tools:node="remove">
 </provider>
```
> 全部移除之后，添加任务需要手动进行初始化配置
```
val configuration = Configuration.Builder()
     .setExecutor(Executors.newFixedThreadPool(2))
     .setMinimumLoggingLevel(android.util.Log.DEBUG)
     .build()
 WorkManager.initialize(this@WorkActivity, configuration)
```



2. 仅移除 WorkManagerInitializer 节点即可。

```
 <provider
    android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    android:exported="false"
    tools:node="merge">
    <!-- If you are using androidx.startup to initialize other components -->
    <meta-data
        android:name="androidx.work.WorkManagerInitializer"
        android:value="androidx.startup"
        tools:node="remove" />
 </provider>
```
> 部分移除后，
 让您的 Application 类实现 Configuration.Provider 接口，并提供您自己的 Configuration.Provider.getWorkManagerConfiguration() 实现。
 当您需要使用 WorkManager 时，请务必调用方法 WorkManager.getInstance(Context)。
 WorkManager 会调用应用的自定义 getWorkManagerConfiguration() 方法来发现其 Configuration。
 （您无需自行调用 WorkManager.initialize()。）
```
class MyApplication() : Application(), Configuration.Provider {
     override fun getWorkManagerConfiguration() =
           Configuration.Builder()
                 .setExecutor(Executors.newFixedThreadPool(2))
                 .setMinimumLoggingLevel(android.util.Log.DEBUG)
                .build()
}
```

# speed 中h5 本地缓存流程
![](https://gitee.com/ZeTing/UploadImg/raw/main/img/%E6%9C%AC%E5%9C%B0H5%20%E4%B8%8B%E8%BD%BD%20%20.png)
