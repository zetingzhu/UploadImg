Android  FileProvider 使用

AndroidManifest.xml 配置文件

```java
<manifest>
    <application>
        <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="${applicationId}.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
         <meta-data
            android:name="android.support.FILE_PROVIDER_PATHS"
            android:resource="@xml/file_paths"/>
        </provider>
    </application>
</manifest>
```

# 属性说明

- provider
  一般为`android.support.v4.content.FileProvider`或者`androidx.core.content.FileProvider`。如果自定义FileProvider，这里则是自定义FileProvider文件的路径（`android:name="com.xxx.XXFileProvider"`）；
  authorities：
  作为唯一标识，用来表明使用者。一般用`包名 + 参数`表示（`android:authorities="${applicationId}.fileprovider"`），在FileProvider的函数`getUriForFile()`方法中需要传入该参数（`APPLICATION_ID + ".fileProvider"`）；
  exported：
  是否允许公开使用；
  grantUriPermissions：
  是否允许授权文件的临时访问权限。
- meta-data
  值为`android.support.FILE_PROVIDER_PATHS`不可修改；
  resource：
  是引用配置文件路径信息的xml文件。

## 在res/xml中创建对外暴露的文件夹路径

① 在项目`app/src/main/res`文件夹下`xml`文件夹，并新建一个`file_paths.xml`文件，编写`file_paths.xml`文件，配置文件路径信息：

```
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
   <external-path name="external_files" path="."/>
</paths>
```

文件路径配置标签可以参考FileProvider文件中的TAG配置：

属性说明：

- paths：
  是一个文件别名，对外可见路径的一部分，隐藏真实文件目录 ；

  path：
  是一个相对目录，相对于当前标签的根目录。`path`值为点符号（"`.`"）时，该根目录下所有的文件夹都可以临时授权访问。

 root-path：
 表示设备的根目录，对应`File DEVICE_ROOT = new File("/")`目录路径：`"/"`；

 files-path：
 表示内部存储空间应用私有目录下的 files/ 目录，对应`Context.getFilesDir()`所获取的目录路径：`/data/data/<包名>/files`；

 cache-path：
 表示内部存储空间应用私有目录下的 cache/ 目录，对应`Context.getCacheDir()`所获取的目录路径：`/data/data/<包名>/cache`；

 external-path：
 表示外部存储空间根目录，对应`Environment.getExternalStorageDirectory()`所获取的目录路径：`/storage/emulate/0`；

 external-files-path：
 表示外部存储空间应用私有目录下的 files/ 目录，对应`Context.getExternalFilesDir()`所获取的目录路径：`/storage/emulate/0/Android/data/<包名>/files`；

 external-cache-path：
 表示外部存储空间应用私有目录下的 cache/ 目录，对应`Context.getExternalCacheDir()`所获取的目录路径：`/storage/emulate/0/Android/data/<包名>/cache`；

 external-media-path：
 表示外部媒体区域根目录中的文件，对应`Context.getExternalMediaDirs()`所获取的目录路径：`/storage/emulated/0/Android/media/<包名>`。



### FileProvider授权使用

1）调用相机

```java
private static void goCamera(Activity activity, String filePath, int requestCode) {
        Intent intent = new Intent();
        File imagePath = new File(filePath);
        Uri imageUri;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
            imageUri = FileProvider.getUriForFile(activity, BuildConfig.APPLICATION_ID + ".fileProvider", imagePath);
            intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
        } else {
            imageUri = Uri.fromFile(imagePath);
        }
        intent.setAction(MediaStore.ACTION_IMAGE_CAPTURE);
        intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
        activity.startActivityForResult(intent, requestCode);
 }
```

2）安装应用

```java
public static void installApk(Context context, String filePath) {
        Intent intent = new Intent(Intent.ACTION_VIEW);
        File file = new File(filePath);
        Uri apkUri;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
            apkUri = FileProvider.getUriForFile(context, BuildConfig.APPLICATION_ID + ".fileProvider", file);
            intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
        } else {
            apkUri = Uri.fromFile(file);
        }
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        intent.setDataAndType(apkUri, "application/vnd.android.package-archive");
        context.startActivity(intent);
 }
```

生成Content Uri文件

```java
File filePath = new File(Context.getFilesDir(), "my_log");
File newFile = new File(filePath, "my_log.log");
// 生成Uri
String authority = context.getPackageName() + ".fileprovider";
Uri contentUri = FileProvider.getUriForFile(getContext(),authority, newFile);
```

这里注意获取目录，在配置paths时我们讲了，paths的子标签必须和获取目录的代码保持对应。这里我们用的是`Context.getFilesDir()`,所以paths文件中必须包含`files-path`子标签，不然别的app获取uri时会出现异常。

最终生成Uri是使用的`FileProvider.getUriForFile()`。第一个参数就是`provider`中设置的`authorities`属性值。
