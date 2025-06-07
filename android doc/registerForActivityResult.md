StartActivityForResult 这是一个通用协定，它可接受任何 `Intent` 作为输入内容并返回 `ActivityResult`，让您能够在回调中提取 `resultCode` 和 `Intent`

RequestMultiplePermissions 用于请求一组权限

RequestPermission 用于请求一个权限

TakePicturePreview 调用MediaStore.ACTION_IMAGE_CAPTURE拍照，返回 `Bitmap`，可以继承它重写createIntent方法传递额外的参数

TakePicture 调用MediaStore.ACTION_IMAGE_CAPTURE拍照，返回`Uri`

TakeVideo  调用MediaStore.ACTION_VIDEO_CAPTURE 拍摄视频，返回`Uri`

PickContact 从通讯录APP获取联系人

GetContent 通过Intent.ACTION_GET_CONTENT选择一条内容，默认添加了Intent.CATEGORY_OPENABLE接收流内容，返回`Uri`

GetMultipleContents 同上，选择多条内容，需要api 18以上

OpenDocument 选择一个文档

OpenMultipleDocuments 选择一个、多个文档

OpenDocumentTree 打开文档树选择文档

CreateDocument 提示用户选择一个路径，创建新文档



 