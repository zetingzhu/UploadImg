Matrix Trace IO Cancry 使用分析

# Trace Canary

- 编译期动态修改字节码, 高性能记录执行耗时与调用堆栈
- 准确的定位到发生卡顿的函数，提供执行堆栈、执行耗时、执行次数等信息，帮助快速解决卡顿问题
- 自动涵盖卡顿、启动耗时、页面切换、慢函数检测等多个流畅性指标
- 准确监控ANR，并且能够高兼容性和稳定性地保存系统产生的ANR Trace文件

# IO Canary

- 接入简单，代码无侵入
- 性能、泄漏全面监控，对 IO 质量心中有数
- 兼容到 Android P

## 使用方法

1. 在你项目根目录下的 gradle.properties 中配置要依赖的 Matrix 版本号，如：

```java
  MATRIX_VERSION=2.0.2
```

1. 在你项目根目录下的 build.gradle 文件添加 Matrix 依赖，如：

```java
  dependencies {
      classpath ("com.tencent.matrix:matrix-gradle-plugin:${MATRIX_VERSION}")
  }
```

1. 接着，在 app/build.gradle 文件中添加 Matrix 各模块的依赖，如：

```java
  dependencies {
implementation "com.tencent.matrix:matrix-android-lib:${MATRIX_VERSION}"
implementation "com.tencent.matrix:matrix-android-commons:${MATRIX_VERSION}"
implementation "com.tencent.matrix:matrix-trace-canary:${MATRIX_VERSION}" 
implementation "com.tencent.matrix:matrix-resource-canary-android:${MATRIX_VERSION}"
implementation "com.tencent.matrix:matrix-resource-canary-common:${MATRIX_VERSION}"
implementation "com.tencent.matrix:matrix-io-canary:${MATRIX_VERSION}"
  }

  apply plugin: 'com.tencent.matrix-plugin'
  matrix {
    trace {
        enable = true	//if you don't want to use trace canary, set false
         println "Monitor project.projectDir:${project.projectDir}"
        baseMethodMapFile = "${project.projectDir}/matrixTrace/methodMapping.txt"
        blackListFile = "${project.projectDir}/matrixTrace/blackMethodList.txt"
    }
  }
```

目前 Matrix gradle plugin 支持 Android Gradle Plugin 3.5.0/4.0.0/4.1.0。

1. 实现 PluginListener，接收 Matrix 处理后的数据, 如：

```java
  public class TestPluginListener extends DefaultPluginListener {
    public static final String TAG = "Matrix.TestPluginListener";
    public TestPluginListener(Context context) {
        super(context);
        
    }

    @Override
    public void onReportIssue(Issue issue) {
        super.onReportIssue(issue);
        MatrixLog.e(TAG, issue.toString());
        
        //add your code to process data
    }
}
```

1. 实现动态配置接口， 可修改 Matrix 内部参数. 在 sample-android 中 我们有个简单的动态接口实例[DynamicConfigImplDemo.java](https://github.com/Tencent/matrix/blob/master/samples/sample-android/app/src/main/java/sample/tencent/matrix/config/DynamicConfigImplDemo.java), 其中参数对应的 key 位于文件 [MatrixEnum](https://github.com/Tencent/matrix/blob/master/samples/sample-android/app/src/main/java/sample/tencent/matrix/config/MatrixEnum.java)中， 摘抄部分示例如下：

```
  public class DynamicConfigImplDemo implements IDynamicConfig {
    public DynamicConfigImplDemo() {}

    public boolean isFPSEnable() { return true;}
    public boolean isTraceEnable() { return true; }
    public boolean isMatrixEnable() { return true; }
    public boolean isDumpHprof() {  return false;}

    @Override
    public String get(String key, String defStr) {
        //hook to change default values
    }

    @Override
    public int get(String key, int defInt) {
         //hook to change default values
    }

    @Override
    public long get(String key, long defLong) {
        //hook to change default values
    }

    @Override
    public boolean get(String key, boolean defBool) {
        //hook to change default values
    }

    @Override
    public float get(String key, float defFloat) {
        //hook to change default values
    }
}
```

1. 选择程序启动的位置对 Matrix 进行初始化，如在 Application 的继承类中， Init 核心逻辑如下：

```java
public void startMatrix(Application application) {
  String[] cpuAbis = Build.SUPPORTED_ABIS;
  Log.d(TAG, "支持的ABIS:" + Arrays.toString(cpuAbis));
 
  Matrix.Builder builder = new Matrix.Builder(application); // build matrix
  builder.pluginListener(new TestPluginListener(this)); // add general pluginListener
  DynamicConfigImplDemo dynamicConfig = new DynamicConfigImplDemo(); // dynamic config
  
 // Configure trace canary.
 TracePlugin tracePlugin = configureTracePlugin(application, dynamicConfig);
 builder.plugin(tracePlugin);

 // Configure io canary.
IOCanaryPlugin ioCanaryPlugin = configureIOCanaryPlugin(dynamicConfig);
builder.plugin(ioCanaryPlugin);
    
  //add to matrix               
  builder.plugin(tracePlugin);
  
  //init matrix
  Matrix.init(builder.build());

  // start plugin 
  tracePlugin.start();
  ioCanaryPlugin.start();
}

// trace canary 配置
private TracePlugin configureTracePlugin(Application application, DynamicConfigImplDemo dynamicConfig) {

        boolean fpsEnable = dynamicConfig.isFPSEnable();
        boolean traceEnable = dynamicConfig.isTraceEnable();
        boolean signalAnrTraceEnable = dynamicConfig.isSignalAnrTraceEnable();

        File traceFileDir = new File(application.getApplicationContext().getFilesDir(), "matrix_trace");
        if (!traceFileDir.exists()) {
            if (traceFileDir.mkdirs()) {
                Log.e(TAG, "failed to create traceFileDir");
            }
        }
        Log.d(TAG, "traceFileDir:" + traceFileDir.getAbsolutePath());

        File anrTraceFile = new File(traceFileDir, "anr_trace");
        // path : /data/user/0/sample.tencent.matrix/files/matrix_trace/anr_trace
        File printTraceFile = new File(traceFileDir, "print_trace");
        // path : /data/user/0/sample.tencent.matrix/files/matrix_trace/print_trace

        Log.d(TAG, "anrTraceFile:" + anrTraceFile.getAbsolutePath());
        Log.d(TAG, "printTraceFile:" + printTraceFile.getAbsolutePath());

        TraceConfig traceConfig = new TraceConfig.Builder()
                .dynamicConfig(dynamicConfig)
                .enableFPS(fpsEnable)
                .enableEvilMethodTrace(traceEnable)
                .enableAnrTrace(traceEnable)
                .enableStartup(traceEnable)
                .enableIdleHandlerTrace(traceEnable) 
                .enableMainThreadPriorityTrace(true) 
                .enableSignalAnrTrace(signalAnrTraceEnable)   
            
                .anrTracePath(anrTraceFile.getAbsolutePath())
            // 通过 SignalAnrTracer.printTrace(); 导入日志到文件
                .printTracePath(printTraceFile.getAbsolutePath())
             .splashActivities("com.trade.eight.moudle.home.activity.LoadingActivity;")
                .isDebug(true)
                .isDevEnv(false)
                .build();

        return new TracePlugin(traceConfig);
    }

// io canary 配置
   private IOCanaryPlugin configureIOCanaryPlugin(DynamicConfigImplDemo dynamicConfig) {
        return new IOCanaryPlugin(new IOConfig.Builder()
                .dynamicConfig(dynamicConfig)
                .build());
    }
```

至此，Matrix就已成功集成到你的项目中



## 变量配置

baseMethodMapFile 配置

```java
matrix {
    trace {
     // 配置为自己环境中的 methodMapping 位置
        baseMethodMapFile = "${project.buildDir}/outputs/mapping/_debugDebug/methodMapping.txt"
        blackListFile = "${project.projectDir}/matrixTrace/blackMethodList.txt"
    }
}

/**
 * 自动将 methodMapping 文件拷贝到移动设备中，方便 matrix 日志分析
 */
project.afterEvaluate {
    tasks.getByName("assemble_debugDebug") {
        it.doLast {
            println 'Monitor 将 mapping 文件拷贝到移动设备中...'
            def sourceFile = new File(project.buildDir.absolutePath + '/outputs/mapping/_debugDebug/methodMapping.txt')
            assert file(sourceFile.absolutePath).exists()
            exec {
                println "Monitor sourceFile: " + sourceFile.absolutePath
                commandLine android.getAdbExecutable(), 'push', "${sourceFile.absolutePath}", '/storage/emulated/0'
            }
        }
    }
}


/**
 * 手动将mapping 文件拷贝到手机中
 */
task copyFileForMethodMapping {
    group = ""
    description = 'Copies methodMapping.txt file from project to device...'
    doFirst {
        println 'Getting file for copy'
        def sourceFile = new File(project.buildDir.absolutePath + '/outputs/mapping/_debugDebug/methodMapping.txt')
        exec {
            println "sourceFile: " + sourceFile.absolutePath
            commandLine android.getAdbExecutable(), 'push', "${sourceFile.absolutePath}", '/storage/emulated/0'
        }
    }
}
```





## 日志打印分析

trace(包括启动、慢函数和 FPS 三种)

### 共同字段：

1. machine: 区分设备好坏的字段
2. process: 进程

### 启动日志分析

1. tag: Trace_StartUp
2. scene: 对应的场景
3. application_create：应用启动的耗时
4. first_activity_create：activity 启动耗时
5. stage_between_app_and_activity: 介于应用和 activity 两者之间的耗时
6. splash_activity_duration，欢迎页耗时
7. startup_duration 启动总耗时
8. is_warm_start_up: 是否是软启动,值范围: true 和 false
9. application_create_scene：启动的场景
   - 100 （activity拉起的）
   - 114（service拉起的）
   - 113 （receiver拉起的）
   - -100 （未知，比如contentprovider）

```java
Monitor.ParseIssueUtil: ISSUE Time Stamp: 2021 年 12 月 09 日 14:53:01:444
    ISSUE TAG: Trace_StartUp
    
    ISSUE Content:
    			machine : MIDDLE
    			cpu_app : 0.0
    			mem : 3972190208
    			mem_free : 1551252
    			application_create : 0
    			application_create_scene : 100
    			first_activity_create : 0
    			startup_duration : 179
    			is_warm_start_up : true
    			tag : Trace_StartUp
    			process : com.rynatsa.xtrendspeed_debug
    			time : 1639032781444
```



### FPS帧率分析

帧率这边需要统计整体帧率，已经按场景统计帧率情况

1. tag: Trace_FPS
2. scene：帧率对应的场景
3. dropLevel：衡量帧率掉帧的水平
4. dropSum：总共掉帧的总时长
5. fps: 帧率

```java
Monitor.ParseIssueUtil: ISSUE Time Stamp: 2021 年 12 月 09 日 14:53:31:570
    ISSUE TAG: Trace_FPS
    
    ISSUE Content:
    			machine : MIDDLE
    			cpu_app : 0.0
    			mem : 3972190208
    			mem_free : 1542424
    			scene : com.trade.eight.moudle.home.activity.MainActivity
    			dropLevel : {"DROPPED_FROZEN":0,"DROPPED_HIGH":2,"DROPPED_MIDDLE":3,"DROPPED_NORMAL":15,"DROPPED_BEST":485}
    			dropSum : {"DROPPED_FROZEN":0,"DROPPED_HIGH":61,"DROPPED_MIDDLE":42,"DROPPED_NORMAL":77,"DROPPED_BEST":54}
    			fps : 42.06930923461914
    			tag : Trace_FPS
    			process : com.rynatsa.xtrendspeed_debug
    			time : 1639032811570

```



### 方法慢解析

对于stack需要通过method_mapping文件解析堆栈，mapping文件在上传安装包的时候需要一起上传。 特殊字段如下：

1. tag: Trace_EvilMethod
2. detail，具体的耗时场景
   a. NORMAL, 普通慢函数场景
   b. ENTER, Activity进入场景
   c. ANR, anr超时场景
   d. FULL, 满buffer场景 e. STARTUP, 启动耗时场景
3. cost: 耗时
4. stack: 堆栈
5. stackKey: 客户端提取的 key，用来标识 issue 的唯一性

如果 detail == ENTER, 会增加viewInfo字段, 包括一下三个属性：

1. viewDeep: view 的深度，是个整数
2. viewCount: view 的数量，是个整数
3. activity: activity 的 name

如果 detail == STARTUP, 会增加subType 字段，默认值-1

- subType=1，代表application初始化过程的堆栈
- subType=2，代表启动第一个界面初始化过程的堆栈

```java
Monitor.ParseIssueUtil: ISSUE Time Stamp: 2021 年 12 月 09 日 15:21:47:831
    ISSUE TAG: Trace_EvilMethod
    ISSUE KEY: 105406017692249
    
    ISSUE Content:
    			machine : MIDDLE
    			cpu_app : 0.0
    			mem : 3972190208
    			mem_free : 1571224
    			detail : ANR
    			cost : 5003
    			stackKey : 185|
    			scene : sample.tencent.matrix.trace.TestTraceMainActivity
    			stack : 0,1048574,1,5003
    1,164,1,5003
    2,170,1,5003
    3,173,1,380
    4,174,1,157
    5,175,1,16
    5,176,1,16
    5,177,1,20
    4,179,1,21
    3,180,1,57
    4,181,1,22
    4,182,1,5
    4,184,1,10
    3,185,1,4566
    
    			threadStack :  
        at android.os.SystemClock:sleep(122)
        at sample.tencent.matrix.trace.TestTraceMainActivity:L(204)
        at sample.tencent.matrix.trace.TestTraceMainActivity:A(150)
        at sample.tencent.matrix.trace.TestTraceMainActivity:testANR(132)
        at java.lang.reflect.Method:invoke(-2)
        at android.view.View$DeclaredOnClickListener:onClick(5363)
        at android.view.View:performClick(6291)
        at android.view.View$PerformClick:run(24931)
        at android.os.Handler:handleCallback(808)
        at android.os.Handler:dispatchMessage(101)
        at android.os.Looper:loop(166)
        at android.app.ActivityThread:main(7529)
    
    			processPriority : 10
    			processNice : -10
    			isProcessForeground : true
    			memory : {"dalvik_heap":12326,"native_heap":18188,"vm_size":2748588}
    			tag : Trace_EvilMethod
    			process : sample.tencent.matrix
    			time : 1639034507831
```



### IO 日志分析

1. tag: io

2. type，耗时这边的类型有两种 a. MAIN_THREAD_IO=1, 在主线程IO超过200ms b. BUFFER_TOO_SMALL=2, 重复读取同一个文件,同一个堆栈超过3次 c. REPEAT_IO=3, 读写文件的buffer过小，即小于4k d. CLOSE_LEAK=4, 文件泄漏

3. path: 文件的路径

4. size: 文件的大小

5. cost: 读写的耗时

6. stack: 读写的堆栈

7. op: 读写的次数

8. buffer: 读写所用的buffer大小，要求大于4k

9. thread: 线程名

10. opType: 1为读，2为写

11. opSize: 读写的总大小

12. repeat:

    a. REPEAT_IO : 重复的次数

    b. Main_IO：1 - 单次操作 2 - 连续读写 3 -2种行为

```java
Monitor.ParseIssueUtil: ISSUE Time Stamp: 2021 年 12 月 09 日 17:22:22:987
    ISSUE TAG: io
    ISSUE Content:
    			path : /sdcard/a_long.txt
    			size : 40960000
    			op : 80000
    			buffer : 512
    			cost : 526
    			opType : 2
    			opSize : 40960000
    			thread : main
    			stack : sample.tencent.matrix.io.TestIOActivity.writeLongSth(TestIOActivity.java:129)
    sample.tencent.matrix.io.TestIOActivity.onClick(TestIOActivity.java:99)
    java.lang.reflect.Method.invoke(Native Method)
    android.view.View$DeclaredOnClickListener.onClick(View.java:5363)
    android.view.View.performClick(View.java:6291)
    android.view.View$PerformClick.run(View.java:24931)
    android.app.ActivityThread.main(ActivityThread.java:7529)
    java.lang.reflect.Method.invoke(Native Method)
    com.android.internal.os.Zygote$MethodAndArgsCaller.run(Zygote.java:245)
    com.android.internal.os.ZygoteInit.main(ZygoteInit.java:921)
    
    			repeat : 0
    			tag : io
    			type : 2
    			process : sample.tencent.matrix
    			time : 1639041742987
```



## stack 解析规则

```java
methodMapping 每行格式：
方法id，方法accessType（Opcodes Access Flags），类名，方法名，方法描述；

stack每行格式： 
stack层级，方法id，方法执行次数，方法执行总耗时。

将两个方法id对比查找对应的堆栈日志
// 下面是堆栈方法解析
String key = "stack";
if (issue.getContent().has(key)) {
    try {
        String stack = issue.getContent().getString(key);
        Map<Integer, String> map = new HashMap<>();
        readMappingFile(map);
        if (map.size() > 0) {
            StringBuilder stringBuilder = new StringBuilder(" ");
            String[] lines = stack.split("\n");
            for (String line : lines) {
                stringBuilder.append("("+line + ")   ");
                String[] args = line.split(",");
                int method = Integer.parseInt(args[1]);
                boolean isContainKey = map.containsKey(method);
                if (!isContainKey) {
                    System.out.print("error!!!");
                    continue;
                }
                args[1] = map.get(method);
                stringBuilder.append(args[0]);
                stringBuilder.append(",");
                stringBuilder.append(args[1]);
                stringBuilder.append(",");
                stringBuilder.append(args[2]);
                stringBuilder.append(",");
                stringBuilder.append(args[3] + "\n");
            }
            issue.getContent().remove(key);
            issue.getContent().put(key, stringBuilder.toString());
        }
    } catch (JSONException ex) {
        System.out.println(ex.getMessage());
    }
}
```



# 问题

1.  
