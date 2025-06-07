
# Matrix for Android
Matrix-android 当前监控范围包括：应用安装包大小，帧率变化，启动耗时，卡顿，慢方法，SQLite 操作优化，文件读写，内存泄漏等等。

## APK Checker:
针对 APK 安装包的分析检测工具，根据一系列设定好的规则，检测 APK 是否存在特定的问题，并输出较为详细的检测结果报告，用于分析排查问题以及版本追踪

## Resource Canary:
基于 WeakReference 的特性和 Square Haha 库开发的 Activity 泄漏和 Bitmap 重复创建检测工具

## Trace Canary:
监控ANR、界面流畅性、启动耗时、页面切换耗时、慢函数及卡顿等问题

## SQLite Lint:
按官方最佳实践自动化检测 SQLite 语句的使用质量

## IO Canary:
检测文件 IO 问题，包括：文件 IO 监控和 Closeable Leak 监控

## Battery Canary:
监控 App 活跃线程（待机状态 & 前台 Loop 监控）、ASM 调用 (WakeLock/Alarm/Gps/Wifi/Bluetooth 等传感器)、 后台流量 (Wifi/移动网络)等 Battery Historian 统计 App 耗电的数据

## MemGuard
检测堆内存访问越界、使用释放后的内存、重复释放等问题



#### APK Checker

- 具有更好的可用性：JAR 包方式提供，更方便应用到持续集成系统中，从而追踪和对比每个 APK 版本之间的变化
- 更多的检查分析功能：除具备 APKAnalyzer 的功能外，还支持统计 APK 中包含的 R 类、检查是否有多个动态库静态链接了 STL 、搜索 APK 中包含的无用资源，以及支持自定义检查规则等
- 输出的检查结果更加详实：支持可视化的 HTML 格式，便于分析处理的 JSON ，自定义输出等等

使用方法：[Matrix Android ApkChecker · Tencent/matrix Wiki (github.com)](https://github.com/Tencent/matrix/wiki/Matrix-Android-ApkChecker)

下载地址 ：https://repo.maven.apache.org/maven2/com/tencent/matrix/matrix-apk-canary/2.0.2/matrix-apk-canary-2.0.2.jar

阿里镜像：https://archiva-maven-storage-prod.oss-cn-beijing.aliyuncs.com/repository/central/com/tencent/matrix/matrix-apk-canary/2.0.2/matrix-apk-canary-2.0.2.jar



1、打包输入apk文件

2、配置修改

```java
{
  //修改打包后的apk路径 输入apk文件路径（默认文件名以apk结尾即可）
  "--apk": "D:/ZZTAndroid/Work_SF/XTrendSF/AppCommon/build/outputs/apk/_debug/release/XTrendSpeed.apk",
  //修改成自己的混淆路径 代码混淆mapping文件路径 （默认文件名是mapping.txt）
  "--mappingTxt": "D:/ZZTAndroid/Work_SF/XTrendSF/AppCommon/build/outputs/mapping/_debugRelease/mapping.txt",
  //修改成自己的输入路径 输出结果文件路径（不含后缀，会根据format决定输出文件的后缀）
  "--output": "D:/ZZTAndroid/Work_SF/XTrendSF/APKChecker/apk-checker-result",

    ......

  "--formatConfig": [
    {
      // 输出结果按照类名(class)或者包名(package)来分组
      "name": "-countMethod",
      "group": [
        {
          "name": "Android System",
          "package": "android"
        },
        {
          "name": "java system",
          "package": "java"
        },
        {
          // 修改成自己的应用包名，方便分组统计dex中方法数
          "name": "com.trade.eight.$",
          "package": "com.trade.eight.$"
        }
      ]
    }
  ],
  "options": [

      ......

    {
      // unusedResources 发现apk中包含的无用资源
      "name":"-unusedResources",
      "--rTxt":"D:/ZZTAndroid/Work_SF/XTrendSF/AppCommon/build/intermediates/runtime_symbol_list/_debugRelease/R.txt",
      "--ignoreResources"
      :["R.raw.*",
        "R.style.*",
        "R.attr.*",
        "R.id.*",
        "R.string.ignore_*"
      ]
    },

      ......

  ]
}
```
```
{
  "--apk":"D:/ZZTAndroid/AndroidUtil/apk-canary/XTrendSpeed_debug.apk",
 "--mappingTxt":"",
  "--resMappingTxt":"",
  "--output":"D:/ZZTAndroid/AndroidUtil/apk-canaryt",
  "--format":"mm.html,mm.json",
  "--formatConfig":
  [
    {
      "name":"-countMethod",
      "group":
      [
        {
          "name":"Android System",
          "package":"android"
        },
        {
          "name":"java system",
          "package":"java"
        },
        {
          "name":"com.trade.eight.$",
          "package":"com.trade.eight.$"
        }
      ]
    }
  ],
  "options": [
    {
      "name":"-manifest"
    },
    {
      "name":"-fileSize",
      "--min":"10",
      "--order":"desc",
      "--suffix":"png, jpg, jpeg, gif, arsc"
    },
    {
      "name":"-countMethod",
      "--group":"package"
    },
    {
      "name":"-checkResProguard"
    },
    {
      "name":"-findNonAlphaPng",
      "--min":"10"
    },
    {
      "name":"-checkMultiLibrary"
    },
    {
      "name":"-uncompressedFile",
      "--suffix":"png, jpg, jpeg, gif, arsc"
    },
    {
      "name":"-countR"
    },
    {
      "name":"-duplicatedFile"
    },

    {
      "name":"-unusedResources",
      "--rTxt":"D:/ZZTAndroid/Work_SF/XTrendSF/AppCommon/build/intermediates/runtime_symbol_list/_debugRelease/R.txt",
      "--ignoreResources"
      :["R.raw.*",
        "R.style.*",
        "R.attr.*",
        "R.id.*",
        "R.string.ignore_*"
      ]
    },
    {
      "name":"-unusedAssets",
      "--ignoreAssets":["*.so" ]
    }

  ]
}
```

3、执行方法

```java
java -jar matrix-apk-canary-2.0.8.jar --config config.json
```

4、结果分析 , 下面是列出这些是可以优化的

​	1. 大图片分析

![image-20211209105447640](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20211209105447640.png)

 	2. 重复文件分析

![image-20211209105531911](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20211209105531911.png)

 	3.  未引用resources文件分析

![image-20211209105740516](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20211209105740516.png)

 	4. 未引用assets 文件分析

![image-20211209110057460](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20211209110057460.png)

​	5. 超大文件排列

![image-20211209110007202](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20211209110007202.png)
