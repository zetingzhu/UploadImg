# 1. 日志颜色设置
~~~
修改路径
File | Settings | Editor | Color Scheme | Android Logcat
jetbrains://AndroidStudio/settings?name=Editor--Color+Scheme--Android+Logcat

Assert:   #FF6B68
Debug:    #0070BB
Error:    #FF0006
Info:     #48BB31
Verbose:  #BBBBBB
Warning:  #BBBB23
~~~

# 2.快捷代码设置
路径
File | Settings | Editor | Live Templates
~~~
sign
private static volatile $NAME$ instance;
public $NAME$() {}
public static  $NAME$  getInstance() {
    if (instance == null) {
        synchronized ( $NAME$ .class) {
            if (instance == null) {
                instance = new  $NAME$ ();
            }
        }
    }
    return instance;
}



ztag
private static final String TAG = $NAME$.class.getSimpleName() ;


ztagk
val TAG = $NAME$::class.java.simpleName


ztest
/**********************测试数据**********************/

/**********************测试数据**********************/

~~~

# 新建项目XML 没有提示
1. 降低 compileSdk 31


# 插件
CamelCase 下划线命名转驼峰
GoogleTranslate 资源文件国际化，反键自动创建
Translation 翻译，可配置翻译源


# 查找依赖树
./gradlew app:dependencies
./gradlew app:dependencies > text

搜索 XXXCompileClasspath、 XXXRunTimeClasspath

./gradlew -q :app:dependencies > textq

./gradlew -q :AppCommon:dependencies > text

./gradlew -q :AppCommon:androidDependencies

./gradlew -q :AppCommon:dependencies --configuration implementation


# 查找列表所有 so 文件位置
```
/**
 * 扫描 so 来源
 */
task findSoFileOrigins {
    description = "扫描项目依赖的 AAR 文件，找出 .so 文件的来源。"
    group = "reporting" // 将任务归类到 "reporting" 组下

    doLast {
        // 用于存储 AAR 标识符及其包含的 .so 文件路径
        // 键 (Key): AAR 的字符串标识符 (例如："project :gsyVideoPlayer", "com.example.library:core:1.0.0")
        // 值 (Value): 一个 Set 集合，包含该 AAR 内所有 .so 文件的路径 (字符串)
        def aarSoFilesMap = [:]

        def variants = null
        if (project.plugins.hasPlugin('com.android.application')) {
            variants = project.android.applicationVariants
        } else if (project.plugins.hasPlugin('com.android.library')) {
            variants = project.android.libraryVariants
        } else {
            project.logger.warn("警告: findSoFileOrigins 任务需要 Android 应用插件 (com.android.application) 或库插件 (com.android.library)。")
            return
        }

        if (variants == null || variants.isEmpty()) {
            project.logger.warn("警告: 未找到任何变体 (variants) 来处理。")
            return
        }

        variants.all { variant ->
            project.logger.lifecycle("正在扫描变体 '${variant.name}' 中的 AAR 依赖以查找 .so 文件...")

            // 获取该变体的运行时配置 (runtime configuration)
            def configuration = variant.getRuntimeConfiguration()

            try {
                // 配置一个构件视图 (artifact view) 来精确请求 AAR 类型的构件
                def resolvedArtifactsView = configuration.incoming.artifactView { view ->
                    view.attributes { attributes ->
                        // 明确指定我们只对 artifactType 为 'aar' 的构件感兴趣
                        // AGP 也常用 "android-aar"，如果 "aar" 效果不佳，可以尝试替换
                        attributes.attribute(Attribute.of("artifactType", String.class), "aar")
                    }
                    // lenient(false) 是默认行为。如果设为 true，它会尝试跳过无法解析的构件而不是让整个视图失败。
                    // 但如果像之前那样，是组件级别的变体选择失败 (如 gsyVideoPlayer)，lenient 可能也无法解决。
                    // view.lenient(false)
                }.artifacts // 获取 ResolvedArtifactSet

                project.logger.info("对于变体 '${variant.name}'，从配置 '${configuration.name}' 解析到 ${resolvedArtifactsView.artifacts.size()} 个 AAR 类型的构件。")

                resolvedArtifactsView.each { resolvedArtifactResult ->
                    // resolvedArtifactResult 是 ResolvedArtifactResult 类型的对象
                    File aarFile = resolvedArtifactResult.file
                    // 获取组件的标识符，这能告诉我们依赖的来源
                    // 例如："project :gsyVideoPlayer" 或 "com.google.android.material:material:1.7.0"
                    String aarIdentifier = resolvedArtifactResult.id.componentIdentifier.displayName

                    aarSoFilesMap.putIfAbsent(aarIdentifier, new HashSet<String>())

                    if (aarFile.exists() && aarFile.name.endsWith('.aar')) {
                        // project.logger.info("正在检查 AAR: ${aarIdentifier} (文件: ${aarFile.name})")
                        try {
                            project.zipTree(aarFile).matching {
                                include '**/*.so' // 匹配 AAR 中的所有 .so 文件
                            }.each { File soFileInZip ->
                                aarSoFilesMap[aarIdentifier].add(soFileInZip.path)
                            }
                        } catch (Exception e) {
                            project.logger.error("错误: 无法检查 AAR 文件 '${aarIdentifier}' (路径: ${aarFile.absolutePath})。原因: ${e.message}")
                        }
                    } else {
                        if (!aarFile.name.endsWith('.aar')) {
                            project.logger.debug("跳过非 AAR 文件 '${aarFile.name}' (来自: ${aarIdentifier})，其构件类型被解析为 AAR。")
                        } else {
                            project.logger.warn("警告: 来自 '${aarIdentifier}' 的 AAR 文件不存在: ${aarFile.absolutePath}")
                        }
                    }
                }

            } catch (Exception e) {
                // 这个 catch 块会捕获解析构件视图时发生的错误
                // 这可能仍然包括之前遇到的 "Could not resolve all artifacts for configuration" 错误，
                // 如果问题非常根本，即使是特定的构件视图也无法克服。
                project.logger.error("错误: 无法为配置 '${configuration.name}' 解析 AAR 类型的构件。" +
                        "可能项目设置中存在依赖变体匹配问题，" +
                        "详细信息: ${e.message}", e) // 打印异常堆栈以获取更多信息
                project.logger.error("建议: 请检查项目依赖（尤其是本地子项目如 ':xxxxx'）的构建配置，" +
                        "确保它们能正确地发布带有标准 Android 库属性（如组件类别、构建类型，以及适用的 Kotlin 平台类型等）的变体。")
                // 如果希望任务在此处停止而不是尝试其他变体，可以取消下一行的注释
                // throw e
            }
        }

        // 打印结果
        if (aarSoFilesMap.isEmpty()) {
            project.logger.lifecycle("\n在所有已处理变体的可解析 AAR 依赖中均未找到 .so 文件，或者依赖解析失败。")
        } else {
            println "\n--- AAR 依赖中的 .so 文件来源 ---"
            // 按 AAR 标识符排序以获得一致的输出
            aarSoFilesMap.sort { it.key }.each { aarId, soFileList ->
                if (!soFileList.isEmpty()) {
                    println "${aarId}:" // 例如：project :gsyVideoPlayer: 或 com.some.library:core:1.0:
                    soFileList.sort().each { soPath -> // 对 .so 文件路径排序
                        println "  - ${soPath}"   // 例如：  - jni/armeabi-v7a/libexample.so
                    }
                }
            }
            println "----------------------------------"
        }
        project.logger.lifecycle("任务执行完毕。要再次运行此任务，请执行: ./gradlew ${project.name}:${name}")
    }
}
```
 
