Android ADB

- 查看设备列表
```
> adb devices
List of devices attached
XPL5T20428011168        offline
192.168.120.168:39871   device

```

- 关闭指定设备
```
adb -s adbName shell reboot -p  
```

- 设备列表出现了 Offline 状态，移除不要设备
```
在命令行或终端中，输入以下命令：
adb kill-server
adb start-server
```

- 查询用户id
```
> adb -s XPL5T20428011168 shell pm list users
Users:
        UserInfo{0:机主:c13} running
        UserInfo{128:分身应用:4000410} running

```

- 终极解决应用分身弹窗问题

> 请将下面命令的xxx换为你分身用户的 ID 分别执行三次ADB命令
该ID是随机的，如何看分身ID命令为 adb shell pm list users
隐私空间同理，这两个功能本质都是多用户
```

adb shell pm disable-user --user xxx com.google.android.gsf
adb shell pm disable-user --user xxx com.google.android.gms
adb shell pm disable-user --user xxx com.android.vending


adb -s XPL5T20428011168 shell pm disable-user --user 4000410 com.google.android.gsf
adb -s XPL5T20428011168 shell pm disable-user --user 4000410 com.google.android.gms
adb -s XPL5T20428011168 shell pm disable-user --user 4000410 com.android.vending

```

- 安装应用
```
> adb -s "25sdfsfb3801745eg" install "C:\Users\joel.joel\Downloads\release.apk"
```
