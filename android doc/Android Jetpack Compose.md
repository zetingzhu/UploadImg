Android Jetpack Compose

# 集成

#### 前提条件

compose 只支持 kotlin 不支持java

#### 开发环境

```kotlin
 implementation("androidx.compose.ui:ui:1.1.1")
    // Tooling support (Previews, etc.)
    implementation("androidx.compose.ui:ui-tooling:1.1.1")
    // Foundation (Border, Background, Box, Image, Scroll, shapes, animations, etc.)
    implementation("androidx.compose.foundation:foundation:1.1.1")
    // Material Design
    implementation("androidx.compose.material:material:1.1.1")
    // Material design icons
    implementation("androidx.compose.material:material-icons-core:1.1.1")
    implementation("androidx.compose.material:material-icons-extended:1.1.1")
    // Integration with observables
    implementation("androidx.compose.runtime:runtime-livedata:1.1.1")
    implementation("androidx.compose.runtime:runtime-rxjava2:1.1.1")

    // UI Tests
    androidTestImplementation("androidx.compose.ui:ui-test-junit4:1.1.1")
```

#### build.gradle 配置信息 ,以下为必须配置信息

```
plugins {
    id 'com.android.application'
    id 'kotlin-android'
}
android {
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = '1.8'
        useIR = true
    }
    buildFeatures {
        compose true
    }
    composeOptions {
        kotlinCompilerExtensionVersion '1.0.1'
        kotlinCompilerVersion '1.5.21'
    }
    packagingOptions {
        resources {
            excludes += '/META-INF/{AL2.0,LGPL2.1}'
        }
    }
}
```

![image-20220628102304770](https://gitee.com/ZeTing/UploadImg/raw/main/img/image-20220628102304770.png)



# 一、控件
## Column
可将多个项垂直地放置在屏幕上

## Row
可将多个项水平地放置在屏幕上

## Box
可将元素放在其他元素上

![Colonm , Row , Box 使用](https://gitee.com/ZeTing/UploadImg/raw/main/img/20221009164810.png)

## Surface 可以自定义风格容器
> 在这里实现了一个带边框圆角和阴影的按钮。Surface 的功能主要有：
- 裁剪，根据 shape 属性描述的形状进行裁剪；
- 高度，根据 elevation 属性设置容器平面的高度，让人看起来有阴影的效果；
- 边框，根据 border 属性设置边框的粗细以及色值；
- 背景，Surface 在 shape 指定的形状上填充颜色。这里会比较复杂一点，如果颜色是 Colors.surface，则会将 LocalElevationOverlay 中设置的 ElevationOverlay 进行叠加，默认情况下只会发生在深色主题中。覆盖的颜色取决于这个 Surface 的高度，以及任何父级 Surface 设置的 LocalAbsoluteElevation。这可以确保一个 Surface 的叠加高度永远不会比它的祖先低，因为它是所有先前 Surface 的高度总和。
- 内容颜色，根据 contentColor 属性给这个平面的内容指定一个首选色值，这个色值会被文本和图标组件以及点击态作为默认色值使用。当然可以被子节点设置的色值覆盖。

```
@Composable
fun SurfaceShow() {
    Surface(
        shape = RoundedCornerShape(6.dp),
        border = BorderStroke(0.5.dp, Color.Green),  // 边框
        elevation = 10.dp,  // 高度
        modifier = Modifier
            .padding(10.dp),  // 外边距
//        color = Color.Black,  // 背景色
        contentColor = Color.Blue,
    ) {
        Surface(
            modifier = Modifier
                .clickable { }  // 点击事件在 padding 前，则此padding为内边距
                .padding(10.dp),
            contentColor = Color.Magenta  // 会覆盖之前 Surface 设置的 contentColor
        ) {
            Text(text = "This is a SurfaceDemo~")
        }
    }
}
```


## Scaffold
基本设计的可视化布局结构，提供了适用于各种组件和其他屏幕元素的槽，
如：顶部应用栏或底部应用栏

```
Scaffold(
          topBar = {
              TopAppBar { /* Top app bar content */ }
          },
          floatingActionButton = {
              FloatingActionButton(onClick = { /* ... */ }) {
                  /* FAB content */
              }
          },
          // Defaults to false
          isFloatingActionButtonDocked = true,
          bottomBar = {
              BottomAppBar { /* Bottom app bar content */ }
          }
      ) { contentPadding ->
          // Screen content

      }
```
## LazyColumn/LazyRow 列表


## Text




# 二、属性
## remember 数据状态保存

## forEachIndexed 循环遍历
```
post.tags.forEachIndexed { index, tag ->
            if (index != 0) {
                append(tagDivider)
            }
            append(" ${tag.uppercase(Locale.getDefault())} ")
        }
```

## buildAnnotatedString  类似于 StringBuffer 工具
```
val text = buildAnnotatedString {
      append(post.metadata.date)
      append(divider)
      append(stringResource(R.string.read_time, post.metadata.readTimeMinutes))
      append(divider)
      post.tags.forEachIndexed { index, tag ->
          if (index != 0) {
              append(tagDivider)
          }
          append(" ${tag.uppercase(Locale.getDefault())} ")
      }
  }
```
```
@Composable
fun MultipleStylesInText() {
    Text(
        buildAnnotatedString {
            withStyle(style = SpanStyle(color = Color.Blue)) {
                append("H")
            }
            append("ello ")

            withStyle(style = SpanStyle(fontWeight = FontWeight.Bold, color = Color.Red)) {
                append("W")
            }
            append("orld")
        }
    )
}
```

# 三、主题

## TopAppBar

# 四、注解
