**Android ASM 使用**














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
