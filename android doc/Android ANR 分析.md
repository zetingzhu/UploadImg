
# Matrix for Android
https://github.com/Tencent/matrix?tab=readme-ov-file#matrix_android_cn

# ANR-WatchDog
https://github.com/SalomonBrys/ANR-WatchDog
# Raster

#   AnrCanary
https://github.com/YanqiangChen/AnrCanary

# Google 阅读文档地址
  https://source.android.google.cn/source/read-bug-reports

# LogCate字段解析



# bugreport 日志查看

## 导出文件
> 1. adb bugreport  输出到当前文件夹下面
> 2. adb bugreport > bugreport.zip

默认情况下，ZIP 文件称为 bugreport-BUILD_ID-DATE.zip，它可能会包含多个文件；
但最重要的文件是 bugreport-BUILD_ID-DATE.txt。此文件就是错误报告，它包含系统服务 (dumpsys)、错误日志 (dumpstate) 和系统消息日志 (logcat) 的诊断输出。
系统消息包括设备抛出错误时的堆栈轨迹，以及从所有应用中使用 Log 类写入的消息。

ZIP 文件中有一个 version.txt 元数据文件，其中包含 Android 版本号，而且启用 systrace 后，ZIP 文件中还会包含 systrace.txt 文件。
Systrace 工具可以获取并显示应用进程和其他 Android 系统进程的执行时间，从而帮助分析应用的性能。

## 搜索文件
> 1. 打开 bugreport.txt 文件，一般这个文件比较大，有15M左右
> 2. 查找 "VM TRACES JUST NOW" 或者 "VM TRACES AT LAST ANR"  往后翻一点就会出现 anr 异常日志信息
ANR in
am_anr



CPU 负载
```
--------- 0.132s was the duration of dumpsys activity, ending at: 2024-04-30 12:36:59
-------------------------------------------------------------------------------
DUMP OF SERVICE CRITICAL cpuinfo:
Load: 5.64 / 6.34 / 5.56
CPU usage from 49124ms to 27093ms ago (2024-04-30 12:36:10.138 to 2024-04-30 12:36:32.169):
  23% 26441/com.rynatsa.xtrendspeed_debug: 18% user + 4.8% kernel / faults: 5586 minor 6 major
  15% 1580/system_server: 10% user + 5.5% kernel / faults: 14249 minor 67 major
  14% 28863/com.android.bluetooth: 10% user + 4.2% kernel / faults: 450 minor 33 major
  10% 29063/com.netease.cloudmusic: 3.9% user + 6.5% kernel / faults: 3956 minor 271 major
  9.8% 855/audioserver: 8.4% user + 1.3% kernel / faults: 475 minor 7 major
  8.9% 27411/com.huawei.appmarket: 6.5% user + 2.4% kernel / faults: 21339 minor 1020 major
  7.9% 29312/com.netease.cloudmusic:play: 4.3% user + 3.6% kernel / faults: 5178 minor 3005 major
  6.4% 859/surfaceflinger: 3% user + 3.3% kernel / faults: 232 minor 4 major
  6.1% 27243/com.alibaba.android.rimet: 3.1% user + 3% kernel / faults: 2527 minor 4 major
  5.1% 821/vendor.huawei.hardware.audio.service: 2% user + 3% kernel / faults: 74 minor
  5.1% 26561/com.huawei.webview:sandboxed_process0:org.chromium.content.app.SandboxedProcessService0:0: 2.9% user + 2.1% kernel / faults: 288 minor
  。。。。。。。。。。
25% TOTAL: 12% user + 9.8% kernel + 0.1% iowait + 2.2% irq + 0.5% softirq
--------- 0.002s was the duration of dumpsys cpuinfo, ending at: 2024-04-30 12:36:59
-------------------------------------------------------------------------------
```
> Load: 5.64 / 6.34 / 5.56
1、5、15 分钟内正在使用和等待使用CPU 的活动进程的平均数

> CPU usage from 49124ms to 27093ms ago (2024-04-30 12:36:10.138 to 2024-04-30 12:36:32.169)
表明负载信息抓取在ANR发生之后的0~1987ms。同时也指明了ANR的时间点：2020-03-10 08:31:55.169

中间部分：各个进程占用的CPU的详细情况

> 25% TOTAL: 12% user + 9.8% kernel + 0.1% iowait + 2.2% irq + 0.5% softirq
各个进程合计占用的CPU信息。

名词解释：
1.user:用户态,kernel:内核态
2.faults:内存缺页，minor——轻微的，major——重度，需要从磁盘拿数据
3.iowait:IO使用（等待）占比
4.irq:硬中断，softirq:软中断



内存信息
```
Total number of allocations 12022134
Total bytes allocated 40GB
Total bytes freed 40GB
Free memory 2260KB
Free memory until GC 2260KB
Free memory until OOME 509MB
Total memory 5251KB
Max memory 512MB
```
Total number of allocations 476778　　进程创建到现在一共创建了多少对象
Total bytes allocated 52MB　进程创建到现在一共申请了多少内存
Total bytes freed 52MB　　　进程创建到现在一共释放了多少内存
Free memory 777KB　　　 不扩展堆的情况下可用的内存
Free memory until GC 777KB　　GC前的可用内存
Free memory until OOME 383MB　　OOM之前的可用内存
Total memory 当前总内存（已用+可用）
Max memory 384MB  进程最多能申请的内存



```
"main" prio=5 tid=1 Runnable
  | group="main" sCount=0 dsCount=0 flags=0 obj=0x73d66ad0 self=0x7d4d6a3a00
  | sysTid=27095 nice=-10 cgrp=default sched=0/0 handle=0x7d526eebb0
  | state=R schedstat=( 44183482862 2730769286 32608 ) utm=3965 stm=453 core=5 HZ=100
  | stack=0x7fc7e9a000-0x7fc7e9c000 stackSize=8MB
  | held mutexes= "mutator lock"(shared held)
```

- 线程自身信息
      线程优先级：prio=5
      线程ID： tid=1
      线程状态：Sleeping
      线程组名称：group="main"
      线程被挂起的次数：sCount=1
      线程被调试器挂起的次数：dsCount=0
      线程的java的对象地址：obj= 0x7682ab30
      线程本身的Native对象地址：self=0x7bd3815c00
- 线程调度信息：
      Linux系统中内核线程ID: sysTid=6317与主线程的进程号相同
      线程调度优先级：nice=-10
      线程调度组：cgrp=default
      线程调度策略和优先级：sched=0/0
      线程处理函数地址：handle= 0x7c59fc8548
- 线程的上下文信息：
      线程调度状态：state=S
      线程在CPU中的执行时间、线程等待时间、线程执行的时间片长度：schedstat=(1009468742 32888019 224)
      线程在用户态中的调度时间值：utm=91
      线程在内核态中的调度时间值：stm=9
      最后执行这个线程的CPU核序号：core=4
- 线程的堆栈信息：
      堆栈地址和大小：stack=0x7ff27e1000-0x7ff27e3000 stackSize=8MB
      线程状态
- 线程的状态很重要，
      了解这些状态的意义对分析ANR的原因是有帮助的，总结如下：

线程状态有5种：新建、就绪、执行、阻塞、死亡
而Java中的线程状态有6种，这6种状态都定义在：java.lang.Thread.State中

|状态|解释 |
|:-------------:|:-----------------:|
NEW|线程刚被创建，但是并未启动。还没调用start方法。
RUNNABLE|线程可以在java虚拟机中运行的状态，可能正在运行自己代码，也可能没有，这取决于操作系统处理器。
BLOCKED|当一个线程试图获取一个对象锁，而该对象锁被其他的线程持有，则该线程进入Blocked状态；当该线程持有锁时，该线程将变成Runnable状态。
WATING|一个线程在等待另一个线程执行一个（唤醒）动作时，该线程进入Waiting状态。进入这个状态后是不能自动唤醒的，必须等待另一个线程调用notify或者notifyAll方法才能够
TIMED_WAITING/TIMEWAITING|同waiting状态，有超时参数，超时后自动唤醒，比如Thread.Sleep(1000)
TERMINATED|线程已经执行完毕


main线程的native是什么状态是CPP代码中定义的状态，下面是一张对应关系表

|Thread.java中定义的状态|Thread.cpp中定义的状态(Anr 日志是这个状态)|说明 |
|:-------------:|:-----------------:|:------------------ |
|TERMINATED     |ZOMBIE             |线程死亡，终止运行
|RUNNABLE       |RUNNING/RUNNABLE   |线程可运行或正在运行
|TIMED_WAITING  |TIMED_WAIT         |执行了带有超时参数的wait、sleep或join函数
|BLOCKED        |MONITOR            |线程阻塞，等待获取对象锁
|WAITING        |WAIT               |执行了无超时参数的wait函数
|NEW            |INITIALIZING       |新建，正在初始化，为其分配资源
|NEW            |STARTING           |新建，正在启动
|RUNNABLE       |NATIVE             |正在执行JNI本地函数
|WAITING        |VMWAIT             |正在等待VM资源
|RUNNABLE       |SUSPENDED          |线程暂停，通常是由于GC或debug被暂停
|               |UNKNOWN            |未知状态


main线程处于 BLOCK 、WAITING 、TIMEWAITING 状态，那基本上是函数阻塞导致ANR；
