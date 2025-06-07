Android Kotlin 升级1.9遇到问题


1. 添加
// 照片选择
   implementation "androidx.activity:activity-ktx:1.9.0"


2.  升级 appcompat
不然没有 registerForActivityResult
api "androidx.appcompat:appcompat:1.6.1"


3.  targetSdkVersion  要改成 34
Dependency 'androidx.activity:activity-ktx:1.9.0' requires libraries and applications that
           depend on it to compile against version 34 or later of the
           Android APIs.

4.



Downloads目录，存放从网络上下载的文件，对应api: MediaStore.Downloads.EXTERNAL_CONTENT_URI
Pictures目录，存图片文件，对应访问api: MediaStore.Images.Media.EXTERNAL_CONTENT_URI
Music目录，存放从网络上下载的文件，对应api: MediaStore.Audio.Media.EXTERNAL_CONTENT_URI
Movies目录，存图片文件，对应访问api: MediaStore.Video.Media.EXTERNAL_CONTENT_UR

Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)



默认图片选择器
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU){
    // 打开Android 13自带的媒体文件选择器（默认单选）
    Intent intent = new Intent(MediaStore.ACTION_PICK_IMAGES);
    // 如果想多选，就加上这行代码，其中MediaStore.getPickImagesMaxLimit()代表设备的Provider最大支持的多选数量
    intent.putExtra(MediaStore.EXTRA_PICK_IMAGES_MAX, Math.min(10,MediaStore.getPickImagesMaxLimit()));
    // 如果只能选择视频，加上这行代码
    intent.setType("video/*");
    // 如果想只选择图片，加上这行代码
    intent.setType("image/*");
    // 如果想限定某种格式的图片或视频，可以把上边的*改成gif或者mp4
    intent.setType("image/gif");
    startActivityForResult(intent, PHOTO_PICKER_MULTI_SELECT_REQUEST_CODE);
}else{
  val intent = Intent(Intent.ACTION_PICK)
intent.type = "image/*"
activity.startActivityForResult(intent, REQUEST_CODE_PICK)
// 选择视频: intent.type = "video/*";
// 选择所有类型的资源: intent.type = "*/*"
}

文件选择
Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
intent.setType("*/*"); // 设置文件类型，这里为任意类型的文件
startActivityForResult(intent, REQUEST_CODE_GET_FILE);

选择文件
val intent = Intent(Intent.ACTION_GET_CONTENT)
intent.type = "*/*"
// 只选择图片: intent.type = "image/*"
// 只选择视频: intent.type = "video/*"

// 支持多选（长按多选）
intent.putExtra(Intent.EXTRA_ALLOW_MULTIPLE, true)

// 用于表示 Intent 仅希望查询能使用 ContentResolver.openFileDescriptor(Uri, String) 打开的 Uri
intent.addCategory(Intent.CATEGORY_OPENABLE)

activity.startActivityForResult(intent, REQUEST_CODE_GET_CONTENT)
// 可以包装 Intent
// activity.startActivityForResult(Intent.createChooser(intent, "选择文件"), REQUEST_CODE_GET_CONTENT)


// 先拿到图片提供者的Uri
val imageUri = MediaStore.Images.Media.EXTERNAL_CONTENT_URI
// 需要获取图片数据表中的哪几列信息，注意这里需要和cursor取出的列名相对应否则会出现空指针
val projection = arrayOf(
    //获取ID列的数据
    MediaStore.Images.Media._ID,
    //获取MIME_TYPE列的值
    MediaStore.Images.Media.MIME_TYPE,
    //获取DISPLAY_NAME列的值
    MediaStore.Images.Media.DISPLAY_NAME
)

// 查询条件：因为是查询全部图片，传null
//String selection = MediaStore.Images.Media.DISPLAY_NAME +"= ?";
// 条件参数：因为是查询全部图片，传null
//String[] args = new String[] {“xxx.png”}
// 排序：可以添加排序
// val order = MediaStore.Files.FileColumns._ID + "DESC"
// 开始查询
val cursor = contentResolver.query(imageUri, projection, null, null, null)

//获取我们需要的数据在数据的第几列，这里需要和projection查询的数据对应起来，因为cursor结果集只包含projection的列信息，否则会出现空指针
if (cursor != null) {
    // 获取id字段是第几列
    val idIndex = cursor.getColumnIndexOrThrow(MediaStore.Images.Media._ID)
    val mimeTypeIndex = cursor.getColumnIndexOrThrow(MediaStore.Images.Media.MIME_TYPE)
    val displayNameIndex = cursor.getColumnIndex(MediaStore.Images.Media.DISPLAY_NAME)

    //循环取出cursor每一行的数据
    while (cursor.moveToNext()) {
        //根据列的坐标，取出对应行数的数据
        val id = cursor.getLong(idIndex)
        //根据列的坐标，取出对应行数的数据
        val type = cursor.getString(mimeTypeIndex)
        //根据列的坐标，取出对应行数的数据
        val disName = cursor.getString(displayNameIndex)
        //根据ID和图片提供者的Uri可以合成图片的Uri
        val imageUri = ContentUris.withAppendedId(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, id)
        //TODO
    }
    //关闭游标
    cursor.close()
}
图片文件比较常见的列名有:

MediaStore.Images.Media._ID：磁盘上文件的路径
MediaStore.Images.Media.DATA：磁盘上文件的路径
MediaStore.Images.Media.DATE_ADDED：文件添加到media provider的时间（单位秒）
MediaStore.Images.Media.DATE_MODIFIED：文件最后一次修改单元的时间
MediaStore.Images.Media.DISPLAY_NAME：文件的显示名称
MediaStore.Images.Media.HEIGHT：图像/视频的高度，以像素为单位
MediaStore.Images.Media.MIME_TYPE：文件的MIME类型
MediaStore.Images.Media.SIZE：文件的字节大小
MediaStore.Images.Media.TITLE：标题
MediaStore.Images.Media.WIDTH：图像/视频的宽度，以像素为单位。
视频文件比较常见的列名有:

MediaStore.Video.Media.TITLE: 名称
MediaStore.Video.Media.DURATION: 总时长
MediaStore.Video.Media.DATA: 地址
MediaStore.Video.Media.SIZE: 大小
MediaStore.Video.Media.WIDTH：视频的宽度，以像素为单位。
MediaStore.Video.Media.HEIGHT：视频的高度，以像素为单位
音频文件比较常见的列名有:

MediaStore.Audio.Media.TITLE：歌名
MediaStore.Audio.Media.ARTIST：歌手
MediaStore.Audio.Media.DURATION：总时长
MediaStore.Audio.Media.DATA：地址
MediaStore.Audio.Media.SIZE：大小




Android系统给我们定义好了许多的媒体文件对应的URI路径如下表:

媒体类型	Uri路径	MediaStore内部类常量	默认存储目录	允许存储目录
Image(图片)	content://media/external/images/media	MediaStore.Images.Media.EXTERNAL_CONTENT_URI	Pictures	DCIM、Pictures
Audio(音频)	content://media/external/audio/media	MediaStore.Audio.Media.EXTERNAL_CONTENT_URI	Music	Alarms、Music、Notifications、Podcasts、Ringtones
Video(视频)	content://media/external/video/media	MediaStore.Video.Media.EXTERNAL_CONTENT_URI	Movies	DCIM 、Movies
Download(下载文件)	content://media/external/downloads	MediaStore.Downloads.EXTERNAL_CONTENT_URI	Download	Download
