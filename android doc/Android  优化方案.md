Android 优化方案

# 冷启动监听

adb 命令查询冷启动时间
> adb命令 : adb shell am start -S -W 包名/启动类的全限定名 ， -S 表示重启当前应用

```java
adb shell am start -S -W com.rynatsa.xtrendspeed_debug/com.trade.eight.moudle.home.activity.LoadingActivity  
Stopping: com.rynatsa.xtrendspeed_debug
Starting: Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] cmp=com.rynatsa.xtrendspeed_debug/com.trade.eight.moudle.home.activity.LoadingActivity }
Status: ok
Activity: com.rynatsa.xtrendspeed_debug/com.trade.eight.moudle.home.activity.LoadingActivity
ThisTime: 2315
TotalTime: 2315
WaitTime: 2369
Complete

```
- ThisTime : 最后一个 Activity 的启动耗时(例如从 LaunchActivity - >MainActivity「adb命令输入的Activity」 , 只统计 MainActivity 的启动耗时)
- TotalTime : 启动一连串的 Activity 总耗时.(有几个Activity 就统计几个)
- WaitTime : 应用进程的创建过程 + TotalTime .

```java
adb shell am start -S -W com.rynatsa.xtrendspeed_debug/com.trade.eight.moudle.home.activity.MainActivity

```

# 内存泄露监听 LeakCanary
https://square.github.io/leakcanary

# Memory Profiler
# MAT
# 卡顿工具 Systrace
