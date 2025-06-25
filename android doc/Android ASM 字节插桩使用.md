**Android ASM 使用**



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










***

编辑环境
Gradle 插件版本 7.4.2
Gradld 版本 7.6.2
![](https://gitee.com/ZeTing/UploadImg/raw/main/img/20231117141150.png)

对应版本官网说明

https://developer.android.google.cn/studio/releases/gradle-plugin?hl=zh-cn#updating-plugin

# Plugin 项目快速创建与调试
为了能够快速调试，并且不用打包打不引用 plugin ，遵循下面几步可以快速编辑调试 plugin
1. 创建一个 javaLib 项目取名 buildSrc
![](https://gitee.com/ZeTing/UploadImg/raw/main/img/20231117140043.png)

2. 去settings.gradle 中删除 include ':buildSrc'
![](https://gitee.com/ZeTing/UploadImg/raw/main/img/20231117140132.png)

3. buildSrc 项目的 build.gradle 添加 plugins ,repositories 和 dependencies
```
plugins {
    id "groovy"
    id "maven-publish"
}
repositories {
    google()
    mavenCentral()
}
dependencies {
    //gradle sdk
    implementation gradleApi()
    //groovy sdk
    implementation localGroovy()
    implementation 'com.android.tools.build:gradle:7.1.3'
    compileOnly("org.ow2.asm:asm-commons:9.3")
}
```
4. 创建 MyPlugin 实现 Plugin<Project>
```
public class MyPlugin implements Plugin<Project> {
    @Override
    public void apply(Project project) {

    }
}
```
5. 在自己的 app 项目中引用这个 plugin
```
import com.example.zzt.buildsrc.MyPlugin
apply plugin: MyPlugin
```


# 简单 ClassVisitor 使用
1. 实现 plugin

```
public class TrackingPluginV3 implements Plugin<Project> {
    private static final String TAG = "ASM-buildSrc-Plugin";

    @Override
    public void apply(Project target) {
        AndroidComponentsExtension comp = target.getExtensions().getByType(AndroidComponentsExtension.class);
        comp.onVariants(comp.selector().all(), new Action<Component>() {
            @Override
            public void execute(Component variant) {
                variant.getInstrumentation().transformClassesWith(TrackingFactoryV3.class, InstrumentationScope.ALL, v -> null);
                variant.getInstrumentation().setAsmFramesComputationMode(FramesComputationMode.COMPUTE_FRAMES_FOR_INSTRUMENTED_METHODS);
            }
        });
    }
}
```

2. 实现 AsmClassVisitorFactory

```
public abstract class TrackingFactoryV3 implements AsmClassVisitorFactory<InstrumentationParameters.None> {
    @Override
    public ClassVisitor createClassVisitor(ClassContext classContext, ClassVisitor classVisitor) {
        return new InsertLogClassVisitor(classVisitor, classContext.getCurrentClassData().getClassName());
    }
    @Override
    public boolean isInstrumentable(@NotNull ClassData classData) {
          return true;
    }
}
```

3. 实现 ClassVisitor , InsertLogClassVisitor 简单实现

```
public class InsertLogClassVisitor extends ClassVisitor {
    private static final String TAG = "ASM-buildSrc-Log";
    String className;

    public InsertLogClassVisitor(ClassVisitor classVisitor, String className) {
        super(Opcodes.ASM5, classVisitor);
        this.className = className;
    }

    @Override
    public MethodVisitor visitMethod(int access, String name, String descriptor, String signature, String[] exceptions) {
        MethodVisitor methodVisitor = super.visitMethod(access, name, descriptor, signature, exceptions);
        AdviceAdapter newMethodVisitor = new AdviceAdapter(Opcodes.ASM5, methodVisitor, access, name, descriptor) {
            @Override
            protected void onMethodEnter() {
                super.onMethodEnter();
                // 方法开始
                mv.visitLdcInsn("TAG");
                mv.visitLdcInsn(className + " -> " + name);
                mv.visitMethodInsn(Opcodes.INVOKESTATIC, "android/util/Log", "i", "(Ljava/lang/String;Ljava/lang/String;)I", false);
                mv.visitInsn(Opcodes.POP);
            }
        };
        return newMethodVisitor;
    }
}
```

# 字节码讲解
  这里不懂，随便截取一个说明一下，方便后面字节码插入

java 代码
```
textView.setOnClickListener(new View.OnClickListener() {
          @Override
          public void onClick(View v) {

          }
      });
```
编辑后 bytecode
```
public void onClick(android.view.View);   // name: onClick
  descriptor: (Landroid/view/View;)V      // descriptor: (Landroid/view/View;)V
  flags: (0x0001) ACC_PUBLIC
  Code:
    stack=0, locals=2, args_size=2
       0: return
    LineNumberTable:
      line 17: 0
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
          0       1     0  this   Lcom/zzt/zztapt/bytecode/JavaBytecode$1;
          0       1     1     v   Landroid/view/View;
// 局部变量  0 this ,1 Landroid/view/View;
```





#   




重复端口占用问题

sudo lsof -i tcp:5005

sudo kill -9 PID

# apt 编译后的文件文件在哪找

./gradlew build -Dorg.gradle.debug=true --no-daemon


opcode 参数：指定了要加载的变量类型。常见的 ALOAD 用于加载引用类型（包括对象和数组）。

Opcodes.ILOAD：加载 int 类型
Opcodes.LLOAD：加载 long 类型
Opcodes.FLOAD：加载 float 类型
Opcodes.DLOAD：加载 double 类型
Opcodes.ALOAD：加载引用类型 (Object/Array)
