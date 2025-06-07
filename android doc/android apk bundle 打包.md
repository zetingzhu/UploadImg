**android apk bundle 打包**

1.根据json配置生成apks包

cmd命令参考如下：

```java
java -jar "D:\ZZTAndroid\Android_bundle_tool\bundletool-all-1.7.0.jar" build-apks --bundle="D:\ZZTAndroid\Android_bundle_tool\intermediary-bundle.aab" --output="D:\ZZTAndroid\Android_bundle_tool\zzttest.apks"  --overwrite --ks="D:\ZZTAndroid\Android_Work\AppFrame5\dev\zztappkey.jks" --ks-pass=pass:android --ks-key-alias=android --key-pass=pass:android --device-spec="D:\ZZTAndroid\Android_bundle_tool\config.json"
```

cmd的命令格式参考如下：

```java
java -jar <bundletool.jar的路径> 
    build-apks 
    --bundle=<.aab文件的路径> 
    --output=<输出.apks的路径> 
    --overwrite 覆盖输出文件
    --ks=<打包.aab文件时的秘钥文件路径，如果.aab文件时没有使用秘钥则可以省去秘钥环节的配置> 
    --ks-pass=pass:<秘钥密码> 
    --ks-key-alias=<秘钥别名> 
    --key-pass=pass:<秘钥别名密码> 
    --device-spec=<要输出的目标sdkVersion的APK的json配置文件路径>
```

json 配置文件参考如下：

```java
{
      "supportedAbis": ["arm64-v8a", "armeabi-v7a"],
      "supportedLocales": ["en", "fr"],
      "screenDensity": 640,
      "sdkVersion": 29
}
```

错误1

```java
Error: Error while reading the device spec file 'D:\Program Files\Android\Android_bundle_tool\config.json'.
java.io.UncheckedIOException: Error while reading the device spec file 'D:\Program Files\Android\Android_bundle_tool\config.json'.
        at com.android.tools.build.bundletool.device.DeviceSpecParser.parseDeviceSpecInternal(DeviceSpecParser.java:64)
        at com.android.tools.build.bundletool.device.DeviceSpecParser.parseDeviceSpec(DeviceSpecParser.java:39)
        at java.util.Optional.map(Unknown Source)
        at com.android.tools.build.bundletool.commands.BuildApksCommand.fromFlags(BuildApksCommand.java:635)
        at com.android.tools.build.bundletool.commands.BuildApksCommand.fromFlags(BuildApksCommand.java:561)
        at com.android.tools.build.bundletool.BundleToolMain.main(BundleToolMain.java:77)
        at com.android.tools.build.bundletool.BundleToolMain.main(BundleToolMain.java:49)
Caused by: com.google.protobuf.InvalidProtocolBufferException: com.google.gson.stream.MalformedJsonException: Expected name at line 4 column 2 path $.supportedLocales
        at com.google.protobuf.util.JsonFormat$ParserImpl.merge(JsonFormat.java:1334)
        at com.google.protobuf.util.JsonFormat$Parser.merge(JsonFormat.java:491)
        at com.android.tools.build.bundletool.device.DeviceSpecParser.parseDeviceSpecInternal(DeviceSpecParser.java:71)
        at com.android.tools.build.bundletool.device.DeviceSpecParser.parseDeviceSpecInternal(DeviceSpecParser.java:61)
        ... 6 more
```

错误2

```java
[BT:1.7.0] Error: Device spec SDK version (0) should be set to a strictly positive number.
com.android.tools.build.bundletool.model.exceptions.InvalidDeviceSpecException: Device spec SDK version (0) should be set to a strictly positive number.
        at com.android.tools.build.bundletool.model.exceptions.UserExceptionBuilder.build(UserExceptionBuilder.java:58)
        at com.android.tools.build.bundletool.device.DeviceSpecParser.validateDeviceSpec(DeviceSpecParser.java:83)
        at com.android.tools.build.bundletool.device.DeviceSpecParser.parseDeviceSpecInternal(DeviceSpecParser.java:73)
        at com.android.tools.build.bundletool.device.DeviceSpecParser.parseDeviceSpecInternal(DeviceSpecParser.java:61)
        at com.android.tools.build.bundletool.device.DeviceSpecParser.parseDeviceSpec(DeviceSpecParser.java:39)
        at java.util.Optional.map(Unknown Source)
        at com.android.tools.build.bundletool.commands.BuildApksCommand.fromFlags(BuildApksCommand.java:635)
        at com.android.tools.build.bundletool.commands.BuildApksCommand.fromFlags(BuildApksCommand.java:561)
        at com.android.tools.build.bundletool.BundleToolMain.main(BundleToolMain.java:77)
        at com.android.tools.build.bundletool.BundleToolMain.main(BundleToolMain.java:49)   
```

错误3

```java
[BT:1.7.0] Error: Flag --output should be the path where to generate the APK Set. Its extension must be '.apks'.
com.android.tools.build.bundletool.model.exceptions.InvalidCommandException: Flag --output should be the path where to generate the APK Set. Its extension must be '.apks'.
        at com.android.tools.build.bundletool.model.exceptions.InternalExceptionBuilder.build(InternalExceptionBuilder.java:57)
        at com.android.tools.build.bundletool.commands.BuildApksCommand$Builder.build(BuildApksCommand.java:543)
        at com.android.tools.build.bundletool.commands.BuildApksCommand.fromFlags(BuildApksCommand.java:643)
        at com.android.tools.build.bundletool.commands.BuildApksCommand.fromFlags(BuildApksCommand.java:561)
        at com.android.tools.build.bundletool.BundleToolMain.main(BundleToolMain.java:77)
        at com.android.tools.build.bundletool.BundleToolMain.main(BundleToolMain.java:49)
```

2.根据连接设备生成apks包
如果您不想针对应用支持的所有设备配置构建一组 APK，则可以使用 --connected-device 选项，仅针对已连接设备的配置生成 APK

```java
java -jar bundletool-all-1.7.0.jar build-apks --bundle=intermediary-bundle.aab --output="d:\Program Files\Android\Android_bundle_tool\zzttest1.apks"   --overwrite --ks="D:\ZZTAndroid\Android_Work\AppFrame5\dev\zztappkey.jks" --ks-pass=pass:android --ks-key-alias=android --key-pass=pass:android --connected-device
```

如果您希望 `bundletool` 只构建一个包含应用的所有代码和资源的 APK

```java
java -jar "D:\ZZTAndroid\Android_bundle_tool\bundletool-all-1.7.0.jar" build-apks --bundle="D:\ZZTAndroid\Android_bundle_tool\intermediary-bundle.aab" --output="D:\ZZTAndroid\Android_bundle_tool\zzttest3.apks"  --overwrite --ks="D:\ZZTAndroid\Android_Work\AppFrame5\dev\zztappkey.jks" --ks-pass=pass:android --ks-key-alias=android --key-pass=pass:android --mode=universal
```

3.安装.apks文件安装到手机
1.插上移动设备（开启调试模式）

2.执行cmd命令

```java
java -jar bundletool-all-1.7.0.jar install-apks --apks="d:\Program Files\Android\Android_bundle_tool\zzttest1.apks"
```
