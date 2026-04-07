# Android 开发文档库 AI 助手指导

这是一个 Android 开发相关的技术文档库，包含了大量 Android 开发实践经验和技术文档。作为 AI 助手，你需要了解以下关键信息来帮助开发者：

## 项目结构

这是一个纯文档库，主要包含 Markdown 格式的技术文档，涵盖以下主要领域：

- Android 基础开发
- Android UI 开发
- Android 性能优化
- Android 打包发布
- Android 新特性适配
- 第三方库使用指南

## 重要技术组件

### 1. Android Studio 和 Gradle

- Gradle Plugin 版本: 7.0.2 - 8.4.1 不等
- Gradle 版本: 7.0.2 - 8.7 不等
- 支持 Java 8 - 17
- Kotlin 1.9+ 支持

### 2. 关键技术框架

- Jetpack 组件
  - Lifecycle
  - WorkManager
  - ViewPager2
- 媒体播放: ExoPlayer
- 依赖注入: Hilt
- UI 框架: 
  - FlexBoxLayout
  - DrawerLayout
  - Compose

### 3. 性能优化

- ASM 字节码插桩
- 应用打包优化 (App Bundle)
- 内存优化 (LruCache)
- 卡顿监控
- 启动优化

## 开发工作流

### 构建和打包

示例 App Bundle 打包命令:
```powershell
java -jar bundletool-all-1.7.0.jar build-apks --bundle="intermediary-bundle.aab" --output="app.apks" --mode=universal
```

### 自定义配置

WorkManager 配置示例:
```kotlin
Configuration.Builder()
    .setExecutor(Executors.newFixedThreadPool(2))
    .setMinimumLoggingLevel(android.util.Log.DEBUG)
    .build()
```

## 注意事项

1. Android 15 升级要点:
   - 升级 AGP 到 8.4.1
   - 升级 Gradle 到 8.7
   - compileSdkVersion 和 targetSdkVersion 升级到 35
   - 添加 namespace 配置

2. ASM 插桩注意:
   - 确保 ASM 版本与 Gradle 版本兼容
   - 使用 `COMPUTE_FRAMES_FOR_INSTRUMENTED_METHODS` 模式

3. ExoPlayer 集成:
   - 最低要求 SDK 34
   - 需要正确配置播放器生命周期

## 常见问题处理

1. 版本兼容性问题:
   - JVM 和 Kotlin 版本匹配
   - Gradle 与 AGP 版本匹配
   - 库依赖版本兼容性

2. 性能问题:
   - 使用 Android Profile 工具分析
   - 配置 ASM 插桩监控性能
   - 应用 LruCache 优化内存使用

3. UI 适配问题:
   - 使用 FlexBoxLayout 做自适应布局
   - 正确处理系统UI（状态栏、导航栏等）

## 相关资源

- Matrix 性能分析工具
- Charles 抓包工具
- Android Profile 性能分析
- Git Submodule 管理