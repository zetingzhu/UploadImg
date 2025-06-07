Xtrend Speed 项目 Android 15 升级

**升级修改**

```java
ext {
  compileSdkVersion = 35
  targetSdkVersion = 35
}

```


# 问题一
Failed to load resources table in APK 'D:\ZZTAndroid\Sdk\platforms\android-35\android.jar'.


**解决方法**

https://planet.kde.org/hanyangzhang-2024-07-23-building-and-running-qml-android-applications/

升级 gradle 8.4.1
 dependencies {
        classpath 'com.android.tools.build:gradle:8.4.1'
    }


---

# 问题二 升级 Gradle

1. 修改
 - 升级 AGP Version   8.4.1
 - 升级 Gradle Version  8.7

解决方法

![](https://gitee.com/ZeTing/UploadImg/raw/main/img/20250123114803.png)

2. 添加 namespace
```
android {
    namespace 'com.easylife.ten.lib'
}
```


## jDK 要升级

Gradle JVM version incompatible.
This project is configured to use an older Gradle JVM that supports up to version 11 but the current AGP requires a Gradle JVM that supports version 17.

**解决方法**

修改jdk 17


## 修改后执行不成功  switchToDebug 任务失败

Reason: Task ':AppCommon:process_debugDebugGoogleServices' uses this output of task ':AppCommon:switchToDebug' without declaring an explicit or implicit dependency. This can lead to incorrect results being produced, depending on what order the tasks are executed.

**解决方法**

<font color=red>暂时没有解决方案，先给去掉</font>

##  错误找不到 tv.danmaku.ijk.media.player 包下类容
错误: 找不到符号
import tv.danmaku.ijk.media.player.IIjkMediaPlayer;

**解决方法**

build.gradle 文件加下添加 aidl 配置
```
android {
/*生成 AIDL 文件对应Class类*/
  buildFeatures{
      aidl true
  }
}
```

![](https://gitee.com/ZeTing/UploadImg/raw/main/img/20250123134248.png)


## 错误 jvm 和 kotlin 版本问题
* What went wrong:
Execution failed for task ':AppCommon:kaptGenerateStubs_debugDebugKotlin'.
> Inconsistent JVM-target compatibility detected for tasks 'compile_debugDebugJavaWithJavac' (1.8) and 'kaptGenerateStubs_debugDebugKotlin' (17).

**解决方法**

```
android {
  compileOptions {
      sourceCompatibility JavaVersion.VERSION_17
      targetCompatibility JavaVersion.VERSION_17
  }

}
```  

---

# 问题三  Transform 已经被删除，无痕埋点需要重新写

### autotrackings 项目以下文件变更


修改文件目录结构
![](https://gitee.com/ZeTing/UploadImg/raw/main/img/20250123164723.png)

- build.gradle

```groovy
apply plugin: 'groovy'
apply plugin: 'maven-publish'

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    //gradle sdk
    implementation gradleApi()
    //groovy sdk
    implementation localGroovy()
    //引入java的gradle api，使用groovy实现则不需要引用
    implementation 'com.android.tools.build:gradle:7.1.3'
    // 引入asm
    compileOnly("org.ow2.asm:asm-commons:9.6")

}
compileGroovy {
    targetCompatibility = JavaVersion.VERSION_17
    sourceCompatibility = JavaVersion.VERSION_17
    options.encoding = "UTF-8"
}

// 上传到本地repo目录
publishing {
    publications {
        maven(MavenPublication) {
            groupId = 'com.jjshome.mobile'
            artifactId = 'tracking-plugin'
            version = '1.5'
            from components.java
        }
    }

    repositories {
        maven {
            url = uri('../repo')
        }
    }
}

```

- tracking.leyoujia.properties

```java
#implementation-class= com.jjshome.mobile.autotrackings.TrackingPlugin
implementation-class= com.jjshome.mobile.autotrackings.TrackingPlugin80
```
- TrackingPlugin80

```java
package com.jjshome.mobile.autotrackings;

import com.android.build.api.instrumentation.FramesComputationMode;
import com.android.build.api.instrumentation.InstrumentationScope;
import com.android.build.api.variant.AndroidComponentsExtension;
import com.android.build.api.variant.Component;

import org.gradle.api.Action;
import org.gradle.api.Plugin;
import org.gradle.api.Project;

/**
 * @author: zeting
 * @date: 2023/10/23
 * 埋点插件
 */
public class TrackingPlugin80 implements Plugin<Project> {
    private static final String TAG = "ASM-Plugin";

    @Override
    public void apply(Project target) {
        System.out.println(TAG + ">>>>>> apply ");
        AndroidComponentsExtension comp = target.getExtensions().getByType(AndroidComponentsExtension.class);
        comp.onVariants(comp.selector().all(), new Action<Component>() {
            @Override
            public void execute(Component variant) {
                variant.transformClassesWith(TrackingFactory80.class, InstrumentationScope.ALL, v -> null);
                variant.setAsmFramesComputationMode(FramesComputationMode.COMPUTE_FRAMES_FOR_INSTRUMENTED_METHODS);
            }
        });
    }
}
```
- TrackingFactory80

```java
package com.jjshome.mobile.autotrackings;

import com.android.build.api.instrumentation.AsmClassVisitorFactory;
import com.android.build.api.instrumentation.ClassContext;
import com.android.build.api.instrumentation.ClassData;
import com.android.build.api.instrumentation.InstrumentationParameters;

import org.objectweb.asm.ClassVisitor;

import groovyjarjarantlr4.v4.runtime.misc.NotNull;


/**
 * @author: zeting
 * @date: 2023/10/23
 * 埋点插件
 */
public abstract class TrackingFactory80 implements AsmClassVisitorFactory<InstrumentationParameters.None> {
    private static final String TAG = "ASM-Factory";

    @Override
    public ClassVisitor createClassVisitor(ClassContext classContext, ClassVisitor classVisitor) {

        System.out.println(TAG + " classContext:" + classContext.getCurrentClassData().toString());
        System.out.println(TAG + " classVisitor:" + classVisitor.toString());

        // 方法中间插入埋点
        return new TrackingClassNode80(classVisitor, classContext.getCurrentClassData().getClassName());
    }


    @Override
    public boolean isInstrumentable(@NotNull ClassData classData) {
        String className = classData.getClassName();
        if (className.startsWith("com.trade.eight.")) {
            return true;
        }
        return false;
    }
}
```

- TrackingClassNode80

```java
package com.jjshome.mobile.autotrackings;


import com.jjshome.mobile.autotrackings.entiy.AnalyticsHookConfig;
import com.jjshome.mobile.autotrackings.entiy.AnalyticsMethodObj;

import org.objectweb.asm.AnnotationVisitor;
import org.objectweb.asm.ClassVisitor;
import org.objectweb.asm.Handle;
import org.objectweb.asm.MethodVisitor;
import org.objectweb.asm.Opcodes;
import org.objectweb.asm.Type;
import org.objectweb.asm.TypePath;
import org.objectweb.asm.commons.AdviceAdapter;

import java.util.HashMap;
import java.util.Map;

/**
 * @author: zeting
 * @date: 2023/10/23
 * 给点击事件添加埋点
 */
public class TrackingClassNode80 extends ClassVisitor {
    private static final String TAG = "ASM-ClassNode";
    private static final String TAG_LAMBDA = "ASM-Lambda";

    // 解析类名字
    private String className;
    /**
     * 存储 Lambda 和方法对应字节码关系
     */
    private Map<String, AnalyticsMethodObj> mLambdaMethodCells = new HashMap<>();

    public TrackingClassNode80(ClassVisitor visitor, String className) {
        super(Opcodes.ASM7, visitor);
        this.className = className;
    }

    @Override
    public void visit(int version, int access, String name, String signature, String superName, String[] interfaces) {
        /**
         * version:52 access:32
         * name:com/zzt/zztapt/act/ActTestJava3$1
         * signature:null
         * superName:java/lang/Object
         * interfaces:[android/view/View$OnClickListener]
         */
        super.visit(version, access, name, signature, superName, interfaces);
    }


    @Override
    public AnnotationVisitor visitTypeAnnotation(int typeRef, TypePath typePath, String descriptor, boolean visible) {
        return super.visitTypeAnnotation(typeRef, typePath, descriptor, visible);
    }

    /**
     * 访问标志，方法名，方法描述符，泛型签名和抛出异常列表，返回一个MethodVisitor
     */
    @Override
    public MethodVisitor visitMethod(int access, String name, String descriptor, String signature, String[] exceptions) {
//        System.out.println(TAG + ">>>>>> visitMethod  access:" + access + " name:" + name + " descriptor:" + descriptor + " signature:" + signature + " exceptions:" + Arrays.toString(exceptions));
        MethodVisitor methodVisitor = super.visitMethod(access, name, descriptor, signature, exceptions);
        AdviceAdapter newMethodVisitor = new AdviceAdapter(Opcodes.ASM7, methodVisitor, access, name, descriptor) {
            /**
             * lambda表达式和方法 使用
             * @param name the method's name.
             * @param descriptor the method's descriptor (see {@link Type}).
             * @param bsm the bootstrap method.
             * @param bsmArgs the bootstrap method constant arguments. Each argument must be
             *     an {@link Integer}, {@link Float}, {@link Long}, {@link Double}, {@link String}, {@link
             *     Type}, {@link Handle} or { ConstantDynamic} value. This method is allowed to modify
             *     the content of the array so a caller should expect that this array may change.
             */
            @Override
            public void visitInvokeDynamicInsn(String name, String descriptor, Handle bsm, Object... bsmArgs) {
                /**
                 * name:onClick
                 * descriptor:()Landroid/view/View$OnClickListener;
                 * bsm:java/lang/invoke/LambdaMetafactory.metafactory(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite; (6)
                 * bsmArgs:[(Landroid/view/View;)V, com/zzt/zztapt/act/ActTestJava3.lambda$onCreate$0(Landroid/view/View;)V (6), (Landroid/view/View;)V]
                 */
                super.visitInvokeDynamicInsn(name, descriptor, bsm, bsmArgs);
//                System.err.println(TAG_LAMBDA + ">>>>>>> visitMethod visitInvokeDynamicInsn name:" + name + " descriptor:" + descriptor + " bsm:" + bsm.toString() + " bsmArgs:" + Arrays.toString(bsmArgs));
                try {
                    String owner = bsm.getOwner();
                    /**
                     * owner: java/lang/invoke/LambdaMetafactory
                     * 代表着 lambda
                     */
                    if (!"java/lang/invoke/LambdaMetafactory".equals(owner)) {
                        return;
                    }
                    /**
                     * desc2: (Landroid/view/View;)V
                     */
                    String desc2 = null;
                    if (bsmArgs[0] instanceof Type) {
                        desc2 = ((Type) bsmArgs[0]).getDescriptor();
                    }
                    /** Landroid/view/View$OnClickListener;onClick(Landroid/view/View;)V
                     *  Desc：Landroid/view/View$OnClickListener;
                     *  name：onClick
                     *  desc2：(Landroid/view/View;)V
                     */
                    String hookKey = Type.getReturnType(descriptor).getDescriptor() + name + desc2;

                    AnalyticsMethodObj sensorsAnalyticsMethodCell = AnalyticsHookConfig.LAMBDA_METHODS.get(hookKey);
                    if (sensorsAnalyticsMethodCell != null) {
                        if (bsmArgs[1] instanceof Handle) {
                            Handle it = (Handle) bsmArgs[1];
                            /**
                             * name:lambda$onCreate$0
                             * desc:(Landroid/view/View;)V
                             */
                            mLambdaMethodCells.put(it.getName() + it.getDesc(), sensorsAnalyticsMethodCell);
                        }
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }

            @Override
            protected void onMethodEnter() {
                addClickEvent(methodVisitor, access, name, descriptor, signature, exceptions);
                addItemClickEvent(methodVisitor, access, name, descriptor, signature, exceptions);
                addSwitchCompatClickEvent(methodVisitor, access, name, descriptor, signature, exceptions);
                addRadioGroupCompatClickEvent(methodVisitor, access, name, descriptor, signature, exceptions);

                addLambdaMethod(methodVisitor, access, name, descriptor, signature, exceptions);
                super.onMethodEnter();
            }

        };
        return newMethodVisitor;
    }

    /**
     * lambda 处理
     *
     * @param mv
     * @param access
     * @param name
     * @param desc
     * @param signature
     * @param exceptions
     */
    public void addLambdaMethod(MethodVisitor mv, int access, String name, String desc, String signature, String[] exceptions) {
        if (name.trim().contains("lambda$")) {
            AnalyticsMethodObj lambdaMethodCell = mLambdaMethodCells.get(name + desc);
            if (lambdaMethodCell != null) {
                Type[] types = Type.getArgumentTypes(lambdaMethodCell.getDesc());
                int length = types.length;
                Type[] lambdaTypes = Type.getArgumentTypes(desc);
                // paramStart 为访问的方法参数的下标，从 0 开始
                int paramStart = lambdaTypes.length - length;
                if (paramStart < 0) {
                    return;
                } else {
                    for (int i = 0; i < length; i++) {
                        if (!lambdaTypes[paramStart + i].getDescriptor().equals(types[i].getDescriptor())) {
                            return;
                        }
                    }
                }
                System.err.println(TAG_LAMBDA + ">>>>>>  add lambda  > className:" + className + " > name:" + (name + desc));

                Boolean isStaticMethod = isStatic(access);
                for (int i = paramStart; i < paramStart + lambdaMethodCell.getParamsCount(); i++) {
                    Integer opCodeA = lambdaMethodCell.getOpcodes().get(i - paramStart);
                    Integer varB = getVisitPosition(lambdaTypes, i, isStaticMethod);
                    mv.visitVarInsn(opCodeA, varB);
                }
                mv.visitMethodInsn(Opcodes.INVOKESTATIC, AnalyticsHookConfig.EVENT_METHOD_CLAZZ, lambdaMethodCell.getAgentName(), lambdaMethodCell.getAgentDesc(), false);
            }
        }
    }

    /**
     * 给 View.OnClick 插入代码
     *
     * @param mv
     * @param name
     * @param descriptor
     */
    public void addClickEvent(MethodVisitor mv, int access, String name, String descriptor, String signature, String[] exceptions) {
        if ("onClick".equals(name) && "(Landroid/view/View;)V".equals(descriptor)) {
            handleViewEventClick(name, mv);
        }
    }

    /**
     * Listview item Click
     *
     * @param mv
     * @param access
     * @param name
     * @param descriptor
     * @param signature
     * @param exceptions
     */
    public void addItemClickEvent(MethodVisitor mv, int access, String name, String descriptor, String signature, String[] exceptions) {
        if ("onItemClick".equals(name) && "(Landroid/widget/AdapterView;Landroid/view/View;IJ)V".equals(descriptor)) {
            handleViewAdapterViewClick(name, mv);
        }
    }

    /**
     * Switch click
     *
     * @param mv
     * @param access
     * @param name
     * @param descriptor
     * @param signature
     * @param exceptions
     */
    public void addSwitchCompatClickEvent(MethodVisitor mv, int access, String name, String descriptor, String signature, String[] exceptions) {
        if ("onCheckedChanged".equals(name) && "(Landroid/widget/CompoundButton;Z)V".equals(descriptor)) {
            handleViewEventClick(name, mv);
        }
    }

    /**
     * Radio Group click
     *
     * @param mv
     * @param access
     * @param name
     * @param descriptor
     * @param signature
     * @param exceptions
     */
    public void addRadioGroupCompatClickEvent(MethodVisitor mv, int access, String name, String descriptor, String signature, String[] exceptions) {
        if ("onCheckedChanged".equals(name) && "(Landroid/widget/RadioGroup;I)V".equals(descriptor)) {
            handleViewEventClick(name, mv);
        }
    }

    /**
     * view点击事件处理
     *
     * @param name
     */
    void handleViewEventClick(String name, MethodVisitor mv) {
        System.err.println(TAG + ">>>>>> add 1 > className:" + className + " > name:" + name);

        mv.visitVarInsn(Opcodes.ALOAD, 1);
        mv.visitMethodInsn(Opcodes.INVOKESTATIC, "com/jjshome/mobile/datastatistics/DSAgent", "onClickView", "(Landroid/view/View;)V", false);
    }


    /**
     * listview ,gridview 列表点击事件
     *
     * @param name
     * @param mv
     */
    void handleViewAdapterViewClick(String name, MethodVisitor mv) {
        System.err.println(TAG + ">>>>>>  add 3  > className:" + className + " > name:" + name);

        mv.visitVarInsn(Opcodes.ALOAD, 1);
        mv.visitVarInsn(Opcodes.ALOAD, 2);
        mv.visitVarInsn(Opcodes.ILOAD, 3);
        mv.visitMethodInsn(Opcodes.INVOKESTATIC, "com/jjshome/mobile/datastatistics/DSAgent", "onAdapterClickView", "(Landroid/widget/AdapterView;Landroid/view/View;I)V", false);
    }


    /**
     * 获取参数类型
     *
     * @param descriptor
     */
    public void getASMMethodType(String name, String descriptor) {
        System.out.println(TAG + "****************************************" + name + "**********************************************");

        Type methodType = Type.getMethodType(descriptor);
        int sizes = methodType.getArgumentsAndReturnSizes();
        System.out.println(TAG + ">>>>>> return type: " + methodType.getReturnType().getClassName() + " (" + (sizes & 3) + ')');

        Type[] argTypes = methodType.getArgumentTypes();
        System.out.println(TAG + ">>>>>> " + argTypes.length + " arguments (" + (sizes >> 2) + ')');
        for (int ix = 0; ix < argTypes.length; ix++) {
            System.out.println(TAG + ">>>>>> arg" + ix + ": " + argTypes[ix].getClassName());
        }

        System.out.println(TAG + "****************************************" + name + "**********************************************");
    }

    /**
     * 获取View 参数位置
     *
     * @param descriptor
     * @return
     */
    public int getASMMethodViewByIndex(String className, String descriptor) {
        try {
            if (isNotNull(className)) {
                Type methodType = Type.getMethodType(descriptor);
                Type[] argTypes = methodType.getArgumentTypes();
                for (int k = 0; k < argTypes.length; k++) {
                    if (className.equals(argTypes[k].getClassName())) {
                        return k;
                    }
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return -1;
    }

    /**
     * 获取方法参数下标为 index 的对应 ASM index
     *
     * @param types          方法参数类型数组
     * @param index          方法中参数下标，从 0 开始
     * @param isStaticMethod 该方法是否为静态方法
     * @return 访问该方法的 index 位参数的 ASM index
     */
    Integer getVisitPosition(Type[] types, Integer index, boolean isStaticMethod) {
        if (types == null || index < 0 || index >= types.length) {
            throw new Error("getVisitPosition error");
        }
        if (index == 0) {
            return isStaticMethod ? 0 : 1;
        } else {
            return getVisitPosition(types, index - 1, isStaticMethod) + types[index - 1].getSize();
        }
    }

    boolean isPrivate(int access) {
        return (access & Opcodes.ACC_PRIVATE) != 0;
    }

    boolean isSynthetic(int access) {
        return (access & Opcodes.ACC_SYNTHETIC) != 0;
    }

    boolean isStatic(int access) {
        return (access & Opcodes.ACC_STATIC) != 0;
    }

    boolean isFinal(int access) {
        return (access & Opcodes.ACC_FINAL) != 0;
    }

    public static boolean isNotNull(String str) {
        return str != null && str.trim().length() > 0;
    }
}

```

- AnalyticsHookConfig

```java
package com.jjshome.mobile.autotrackings.entiy;

import org.objectweb.asm.Opcodes;

import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * @author: zeting
 * @date: 2023/11/13
 * 配置的分析信息
 */
public class AnalyticsHookConfig {
    /**
     * 分析埋点插入类
     */
    public static String EVENT_METHOD_CLAZZ = "com/jjshome/mobile/datastatistics/DSAgent";

    public static Map<String, AnalyticsMethodObj> LAMBDA_METHODS = new HashMap<>();

    static {
        addLambdaMethod(
                new AnalyticsMethodObj(
                        "onClick",
                        "(Landroid/view/View;)V",
                        "Landroid/view/View$OnClickListener;",
                        "onClickView",
                        "(Landroid/view/View;)V",
                        1, 1,
                        List.of(Opcodes.ALOAD))
        );
        addLambdaMethod(
                new AnalyticsMethodObj(
                        "onCheckedChanged",
                        "(Landroid/widget/CompoundButton;Z)V",
                        "Landroid/widget/CompoundButton$OnCheckedChangeListener;",
                        "onClickView",
                        "(Landroid/view/View;)V",
                        1, 1,
                        List.of(Opcodes.ALOAD)
                )
        );
        addLambdaMethod(
                new AnalyticsMethodObj(
                        "onCheckedChanged",
                        "(Landroid/widget/RadioGroup;I)V",
                        "Landroid/widget/RadioGroup$OnCheckedChangeListener;",
                        "onClickView",
                        "(Landroid/view/View;)V",
                        1, 1,
                        List.of(Opcodes.ALOAD)
                )
        );
        addLambdaMethod(
                new AnalyticsMethodObj(
                        "onItemClick",
                        "(Landroid/widget/AdapterView;Landroid/view/View;IJ)V",
                        "Landroid/widget/AdapterView$OnItemClickListener;",
                        "onAdapterClickView",
                        "(Landroid/widget/AdapterView;Landroid/view/View;I)V",
                        1, 3,
                        Arrays.asList(Opcodes.ALOAD, Opcodes.ALOAD, Opcodes.ILOAD)
                )
        );
    }

    private static void addLambdaMethod(AnalyticsMethodObj sensorsAnalyticsMethodCell) {
        if (sensorsAnalyticsMethodCell != null) {
            LAMBDA_METHODS.put(
                    sensorsAnalyticsMethodCell.getParent() + sensorsAnalyticsMethodCell.getName() + sensorsAnalyticsMethodCell.getDesc(),
                    sensorsAnalyticsMethodCell
            );
        }
    }
}

```

- AnalyticsMethodObj

```java
package com.jjshome.mobile.autotrackings.entiy;

import java.util.List;
import java.util.Objects;

/**
 * @author: zeting
 * @date: 2023/11/13
 * 分析信息参数
 */
public class AnalyticsMethodObj {
    /**
     * 方法名字
     */
    private String name;
    /**
     * 方法参数和返回值描述
     */
    private String desc;

    /**
     * 方法所在的接口或类
     */
    private String parent;

    /**
     * 调用采集数据方法
     */
    private String agentName;

    /**
     * 采集数据的方法描述
     */
    private String agentDesc;

    /**
     * 采集数据的方法参数起始索引（ 0：this，1+：普通参数 ）
     */
    private Integer paramsStart;

    /**
     * 采集数据的方法参数个数
     */
    private Integer paramsCount = 0;

    /**
     * 参数类型对应的ASM指令，加载不同类型的参数需要不同的指令
     */
    private List<Integer> opcodes;

    public AnalyticsMethodObj(String name, String desc, String parent, String agentName, String agentDesc, Integer paramsStart, Integer paramsCount, List<Integer> opcodes) {
        this.name = name;
        this.desc = desc;
        this.agentName = agentName;
        this.parent = parent;
        this.agentDesc = agentDesc;
        this.paramsStart = paramsStart;
        this.paramsCount = paramsCount;
        this.opcodes = opcodes;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDesc() {
        return desc;
    }

    public void setDesc(String desc) {
        this.desc = desc;
    }

    public String getAgentName() {
        return agentName;
    }

    public void setAgentName(String agentName) {
        this.agentName = agentName;
    }

    public String getParent() {
        return parent;
    }

    public void setParent(String parent) {
        this.parent = parent;
    }

    public String getAgentDesc() {
        return agentDesc;
    }

    public void setAgentDesc(String agentDesc) {
        this.agentDesc = agentDesc;
    }

    public Integer getParamsStart() {
        return paramsStart;
    }

    public void setParamsStart(Integer paramsStart) {
        this.paramsStart = paramsStart;
    }

    public Integer getParamsCount() {
        return paramsCount;
    }

    public void setParamsCount(Integer paramsCount) {
        this.paramsCount = paramsCount;
    }

    public List<Integer> getOpcodes() {
        return opcodes;
    }

    public void setOpcodes(List<Integer> opcodes) {
        this.opcodes = opcodes;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof AnalyticsMethodObj)) return false;
        AnalyticsMethodObj that = (AnalyticsMethodObj) o;
        return Objects.equals(getName(), that.getName()) && Objects.equals(getDesc(), that.getDesc()) && Objects.equals(getAgentName(), that.getAgentName()) && Objects.equals(getParent(), that.getParent()) && Objects.equals(getAgentDesc(), that.getAgentDesc()) && Objects.equals(getParamsStart(), that.getParamsStart()) && Objects.equals(getParamsCount(), that.getParamsCount()) && Objects.equals(getOpcodes(), that.getOpcodes());
    }

    @Override
    public int hashCode() {
        return Objects.hash(getName(), getDesc(), getAgentName(), getParent(), getAgentDesc(), getParamsStart(), getParamsCount(), getOpcodes());
    }

    @Override
    public String toString() {
        return "SensorsAnalyticsMethodCell{" +
                "name='" + name + '\'' +
                ", desc='" + desc + '\'' +
                ", agentName='" + agentName + '\'' +
                ", parent='" + parent + '\'' +
                ", agentDesc='" + agentDesc + '\'' +
                ", paramsStart=" + paramsStart +
                ", paramsCount=" + paramsCount +
                ", opcodes=" + opcodes +
                '}';
    }
}

```

修改文件后，执行一下 Task.publish

![](https://gitee.com/ZeTing/UploadImg/raw/main/img/20250123162155.png)

会生成一个 1.5 插件
![](https://gitee.com/ZeTing/UploadImg/raw/main/img/20250123162337.png)

修改根目录下的 build.gradle ，替换插件版本号
![](https://gitee.com/ZeTing/UploadImg/raw/main/img/20250123162434.png)



## 错误如果用到下面错误，
```
Execution failed for task ':AppCommon:transform_debugDebugClassesWithAsm'.
> A failure occurred while executing com.android.build.gradle.tasks.TransformClassesWithAsmTask$TransformClassesIncrementalAction
   > Error occurred while instrumenting class com.trade.eight.moudle.mission.vm.base.LoadUiState

```

**解决方法**
直接把 LoadUiState 文件给删除，目前没有用



# 问题四
后续问题
1. Echat 聊天页面没有适配，会跑到导航栏下面去
![](https://gitee.com/ZeTing/UploadImg/raw/main/img/20250123184117.png)
