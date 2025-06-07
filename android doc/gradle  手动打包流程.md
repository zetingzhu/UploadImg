**gradle  手动打包流程** 

1. AAPT2 compile 资源
   打包资源文件生产res压缩包
   -> D:\ZZTAndroid\Sdk\build-tools\30.0.2\aapt2 compile -o tmp/res --dir simpleapk/src/main/res/

解压资源文件
-> unzip -u tmp/res -d tmp/aapt2_res
Archive:  tmp/res
 extracting: tmp/aapt2_res/drawable-v24_ic_launcher_foreground.xml.flat
 extracting: tmp/aapt2_res/drawable_ic_launcher_background.xml.flat
 extracting: tmp/aapt2_res/layout_activity_simple_apk.xml.flat
 extracting: tmp/aapt2_res/mipmap-anydpi-v26_ic_launcher.xml.flat
 extracting: tmp/aapt2_res/mipmap-anydpi-v26_ic_launcher_round.xml.flat
 extracting: tmp/aapt2_res/mipmap-hdpi_ic_launcher.png.flat
 extracting: tmp/aapt2_res/mipmap-hdpi_ic_launcher_round.png.flat
 extracting: tmp/aapt2_res/mipmap-mdpi_ic_launcher.png.flat
 extracting: tmp/aapt2_res/mipmap-mdpi_ic_launcher_round.png.flat
 extracting: tmp/aapt2_res/mipmap-xhdpi_ic_launcher.png.flat
 extracting: tmp/aapt2_res/mipmap-xhdpi_ic_launcher_round.png.flat
 extracting: tmp/aapt2_res/mipmap-xxhdpi_ic_launcher.png.flat
 extracting: tmp/aapt2_res/mipmap-xxhdpi_ic_launcher_round.png.flat
 extracting: tmp/aapt2_res/mipmap-xxxhdpi_ic_launcher.png.flat
 extracting: tmp/aapt2_res/mipmap-xxxhdpi_ic_launcher_round.png.flat
 extracting: tmp/aapt2_res/values_colors.arsc.flat
 extracting: tmp/aapt2_res/values_strings.arsc.flat
 extracting: tmp/aapt2_res/values_themes.arsc.flat

2. AAPT2 link 资源
   AAPT2 link 是把上一步 compile 处理后的 xxx.flat 资源链接，生成一个完整的 resource.arsc，二进制资源和 R.java
   -> D:\ZZTAndroid\Sdk\build-tools\30.0.2\aapt2 link -o tmp/res.apk  -I D:\ZZTAndroid\Sdk\platforms\android-30\android.jar --manifest simpleapk/src/main/AndroidManifest.xml  --java tmp   -R tmp/aapt2_res/drawable-v24_ic_launcher_foreground.xml.flat -R tmp/aapt2_res/drawable_ic_launcher_background.xml.flat  -R tmp/aapt2_res/layout_activity_simple_apk.xml.flat  -R tmp/aapt2_res/mipmap-anydpi-v26_ic_launcher.xml.flat  -R tmp/aapt2_res/mipmap-anydpi-v26_ic_launcher_round.xml.flat  -R tmp/aapt2_res/mipmap-hdpi_ic_launcher.png.flat  -R tmp/aapt2_res/mipmap-hdpi_ic_launcher_round.png.flat  -R tmp/aapt2_res/mipmap-mdpi_ic_launcher.png.flat  -R tmp/aapt2_res/mipmap-mdpi_ic_launcher_round.png.flat  -R tmp/aapt2_res/mipmap-xhdpi_ic_launcher.png.flat  -R tmp/aapt2_res/mipmap-xhdpi_ic_launcher_round.png.flat  -R tmp/aapt2_res/mipmap-xxhdpi_ic_launcher.png.flat  -R tmp/aapt2_res/mipmap-xxhdpi_ic_launcher_round.png.flat  -R tmp/aapt2_res/mipmap-xxxhdpi_ic_launcher.png.flat  -R tmp/aapt2_res/mipmap-xxxhdpi_ic_launcher_round.png.flat  -R tmp/aapt2_res/values_colors.arsc.flat  -R tmp/aapt2_res/values_strings.arsc.flat  -R tmp/aapt2_res/values_themes.arsc.flat --auto-add-overlay

执行命令后，会生成 res.apk，里面就是 resource.arsc，处理后的 AndroidManifest.xml 以及 处理后的二进制资源。我们这里也把他解压出来，后面最终打包的时候使用。
-> unzip -u tmp/res.apk -d tmp/final

3. javac 生成 class 文件
   javac 生成 class 文件
   -> javac -d tmp simpleapk\src\main\java\com\zzt\simpleapk\ActivitySimpleApk.java simpleapk\src\main\java\com\zzt\simpleapk\ABC.java  tmp\com\zzt\simpleapk\R.java  -cp  D:\ZZTAndroid\Sdk\platforms\android-30\android.jar

4. d8 编译 dex
   这一步是把上一步生成的 class 文件编译为 dex 文件，需要用到 d8 或者 dx，这里用 d8。执行下面的命令。
   -> D:\ZZTAndroid\Sdk\build-tools\30.0.2\d8 tmp\com\zzt\simpleapk\*.class --output tmp\final --lib D:\ZZTAndroid\Sdk\platforms\android-30\android.jar

5. 打包 apk
   执行完上述的命令，打包 APK 需要的材料就都准备好了，因为 APK 本身就是 zip 格式，这里我们直接用 zip 命令打包上述产物，生成 final.apk。执行下面的命令。
   进入到指定的文件目录
   ->  cd tmp/final
   压缩成apk文件
   ->  zip -r ..\final.apk *

6. apk 签名
   上一步打包好的 APK 还不能直接安装，因为没有签名，我们这里用 debug.keystore 给 final.apk 签名。执行下面的命令。
   ->  D:\ZZTAndroid\Sdk\build-tools\30.0.2\apksigner sign --ks C:\Users\KTing\.android\debug.keystore tmp\final.apk
   复制代码这里需要输入 debug.keystore 的密码，是 android。

7.安装apk
-> adb install -r tmp\final.apk