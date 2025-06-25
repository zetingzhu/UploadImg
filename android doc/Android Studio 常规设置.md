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
