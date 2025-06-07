Android 字节插桩

# 1. 当前环境

## Gradle Plugin Version

classpath "com.android.tools.build:gradle:7.0.2"

## Gradle Version

distributionUrl=https\://services.gradle.org/distributions/gradle-7.0.2-bin.zip

# 2. 创建步骤

新建一个module，除了build.gradle文件和 src/main 包名其他都删除

build.gradle 配置文件

```
apply plugin: 'groovy'  
apply plugin: 'maven-publish'

dependencies {  
 implementation gradleApi()  
 implementation localGroovy()  
 implementation 'com.android.tools.build:gradle:3.4.1'  
}  
publishing {  
 publications {  
 maven(MavenPublication) {  
// maven依赖关系  classpath "com.zzt:transform:1.0.0"
         groupId = 'com.zzt'
            artifactId = 'transform'
            version = '1.0.0'
 from components.java  
 }  
 } repositories {  
 maven {  
 // maven 本地保存路径
 url = '../repo'  
 }  
 }}
```

新建一个包resources,内部新建一个包 META-INF/gradle-plugins

在创建文件 properties 文件

文件名为com.zzt.transform.properties，

这个文件名使用插件，就可以这样：apply plugin 'com.zzt.transform'；

起完名字后还不可以使用该插件，还要告诉Gradle自定义插件的具体实现类是哪一个，在com.zzt.transform.properties文件中添加如下内容

```
implementation-class=com.zt.gradle.ztTransform
```

在src/main 下新建一个包名groovy

正常的新建一个groovy文件

com.zt.gradle/ZTTransform.groovy

此文件实现Plugin接口

![](https://gitee.com/ZeTing/UploadImg/raw/main/img/微信截图_20220412211317.png)

编辑成插件，选中右边gradle 找到创建项目的 publish 执行

![](https://gitee.com/ZeTing/UploadImg/raw/main/img/微信截图_20220412212718.png)

# 3.使用方法

1. 添加本地依赖
   
   ```
       // 添加Maven的本地依赖
           maven {
               url uri('./repo')
           }
   ```

2. 添加依赖库

```
    classpath "com.zzt:transform:1.0.0"
```

3. 在依赖项目中添加插件

```
plugins {
// 这个就是 properties 文件名称
    id 'com.zzt.transform'
}
```
