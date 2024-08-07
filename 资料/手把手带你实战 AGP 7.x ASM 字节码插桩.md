---
author: zjmantou
title: 手把手带你实战 AGP 7.x ASM 字节码插桩
time: 2024-08-06 周二
tags:
  - 资料
  - Gradle
  - ASM
---
![](https://static001.geekbang.org/infoq/cd/cd9cf600e08599924514884a71c25f4b.png)

## 一、前言

字节码插桩技术在 Android 领域应用广泛，甚至在不少中高级面试中，是必问的技术面之一。它的应用场景包括但不限于：

- 性能优化：监控函数耗时，优化线程数量。
    
- 无痕埋点：不侵入业务源码，实现全量埋点。
    
- 隐私合规：监控敏感方法调用，防止 App 因安全风险等原因而被下架。
    
- ……
    

字节码插桩的本质是**对字节码文件（.class）的修改**。从原理上讲，利用文本编辑器手动编辑也是能修改的（笑，但实际上一般是通过各种框架来做。实现字节码插桩的框架有很多，

- ASM：[https://asm.ow2.io/](https://asm.ow2.io/ "https://asm.ow2.io/")
    
- AspectJ：[https://www.eclipse.org/aspectj/](https://www.eclipse.org/aspectj/ "https://www.eclipse.org/aspectj/")
    
- ReDex：[https://fbredex.com/](https://fbredex.com/ "https://fbredex.com/")
    

在 AGP 7.0 以前，通过 AGP 的 Transform API 来实现字节码插桩；从 7.0 开始，Transform API 被声明为 Deprecated，并计划在 AGP 的 8.0 版本中移除。但这并不表示无法再使用字节码插桩了，相反，有一套新的 API —— TransformAction，供我们实现这一需求。

## 二、目的

一句话，本文将会带大家一步一步使用 AGP 7.0 开始推荐使用的 TransformAction API，来实现 ASM 插桩。

## 可以学到什么

- 如何编写一个 Gradle Plugin
    
- 如何使用 TransformAction API 进行字节码插桩
    
- 了解其中的一些坑，避免重蹈覆辙
    

## 需要了解什么

- Kotlin 语法
    
- Gradle 的一些知识：包括不限于文件组成结构、Composite Build
    

## 三、实战

有时我们想知道一个函数究竟有没有被执行，常见的手段有断点、手动加 Log。今天我们以 ASM 插桩的方式，在进入函数时打印一条 Log 和时间点。

假设已经有一个 Android 项目（可以新建一个），并且确保 AGP 版本大于 7.0.0。下面所用到的 Android 项目（主工程）名称为 AsmTourism。

## 1. 添加自定义插件工程目录

Gradle 7 引入了 Composite Build，可以让一个 Gradle 项目参与到另一个 Gradle 项目的构建当中。这里采用这种方式来实现自定义插件，而不是使用保留目录 `buildSrc`（会有坑）或者发布插件的方式。

### 1.1 新建 `build-logic` 目录

在项目根目录下新建 `build-logic` 文件夹（名字其实可以随意），并在该目录内创建 `settings.gradle.kts` 文件。

再把主工程的 `gradle` 文件夹、`gradle.properties` 文件拷贝到该目录。

此时的目录结构如下：

![](https://static001.geekbang.org/infoq/fc/fc67518df0785a73188faf412a9f6cd2.png)

新建 build-logic 后，主工程的文件结构

### 1.2 新建 `convention` 目录

**在 build-logic 目录下**，新建 `convention` 目录，作为插件源码的模块目录。

同时，还需要创建 Gradle 项目必备的文件 `convention/build.gradle.kts` 和文件夹 `convention/src/main/kotlin/`。

此时的目录结构如下：

![新建 convention module 后，build-logic 工程的文件结构](file:///Users/mantou/.config/joplin-desktop/resources/832699b283b8435fae4ca6de407db246.png)

### 1.3 连接主工程和 build-logic 工程

在主工程的 `settings.gradle` 文件中，使用 `includeBuild` 将主工程 `AsmTourism` 和插件工程 `build-logic` 连接起来。

```kotlin
 pluginManagement {
+    includeBuild("build-logic")
     repositories {
         gradlePluginPortal()
         google()
```

### 1.4 小结

熟悉 Gradle 的朋友会发现，这里其实相当于是在主工程目录中，创建了一个 Gradle 工程，它有它自己的 `settings.gradle.kts` 文件和一个名为 `convention` 的模块。

> Tips：如果一个目录下存在 settings.gradle.kts 文件，Gradle 会把它当作一个 Gradle 工程，而不是模块。

## 2. 编写 Gradle 插件

### 2.1 配置 `build-logic/settings.gradle.kts`

```kotlin
// 配置项目的依赖源
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
    }
}
// 将 convention 模块加入编译
include(":convention")
```


### 2.2 配置 `convention/build.gradle.kts`

```kotlin
plugins {
  	// 使用 kotlin dsl 作为 gradle 构建脚本语言
    `kotlin-dsl`
}

// 配置字节码的兼容性
java {
    sourceCompatibility = JavaVersion.VERSION_1_8
    targetCompatibility = JavaVersion.VERSION_1_8
}

dependencies {
    val agpVersion = "7.2.2"
    val kotlinVersion = "1.7.10"
    val asmVersion = "9.3"
  	// AGP 依赖
    implementation("com.android.tools.build:gradle:$agpVersion") {
        exclude(group = "org.ow2.asm")
    }
    // Kotlin 依赖 —— 插件使用 Kotlin 实现
    implementation("org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlinVersion") {
        exclude(group = "org.ow2.asm")
    }
    // ASM 依赖库
    implementation("org.ow2.asm:asm:$asmVersion")
    implementation("org.ow2.asm:asm-commons:$asmVersion")
    implementation("org.ow2.asm:asm-util:$asmVersion")
}

gradlePlugin {
    plugins {
      	// 注册插件，这样可以在其他地方 apply
        register("LogPlugin") {
          	// 注册插件的 id，需要应用该插件的模块可以通过 apply 这个 id
            id = "me.hjhl.gradle.plugin.log"
            implementationClass = "LogPlugin"
        }
    }
}
```


### 2.3 创建插件 `LogPlugin`

在 `build-logic/convention/src/main/kotlin/` 目录下新建 `LogPlugin.kt` 文件，内容如下：

```kotlin
import org.gradle.api.Plugin
import org.gradle.api.Project

class LogPlugin : Plugin<Project> {
    companion object {
        private const val TAG = "LogPlugin"
    }

    override fun apply(target: Project) {
        log("======== start apply ========")
        log("apply target: ${target.displayName}")
        log("========  end apply  ========")
    }

    private fun log(msg: String) {
        println("[$TAG]: $msg")
    }
}
```

此时的目录结构如下：

![添加插件源文件后的文件结构](https://static001.geekbang.org/infoq/99/9944c757d0e05ce684dbb72f97cbc193.png)



### 2.4 `app` 模块中应用插件

回到 `app/build.gradle` 文件，在 `plugins` 语句块中应用该插件，如下：

```kotlin
 plugins {
     id 'com.android.application'
     id 'org.jetbrains.kotlin.android'
     id 'me.hjhl.gradle.plugin.log'
 }

 android {
```

sync 一下工程，如果可以在 AS 的 Build 窗口中如下输出：

![](https://static001.geekbang.org/infoq/00/005dbbf0a67fa7cbf80999a0972c4a26.png)

则表示插件应用成功了。

### 2.5 小节

这一步需要重点注意插件在项目中的注册和使用。其次，需要熟悉下 Gradle 插件的编写方式 —— 从继承 `Plugin<Project>` 开始。

## 3. 实现 ASM 插桩

### 3.1 编写 Transform 类，实现 ClassVisitor

不需要繁琐的手段，AGP 提供了一个抽象接口 `AsmClassVisitorFactory` 简化了 Transform 的编写流程，我们只需要这样使用：

1. **定义一个抽象类**，实现该接口。
    
2. 实现 `createClassVisitor` 和 `isInstrumentable` 两个方法。
    

如下：

```kotlin
package me.hjhl.gradle.plugin.log

import com.android.build.api.instrumentation.AsmClassVisitorFactory
import com.android.build.api.instrumentation.ClassContext
import com.android.build.api.instrumentation.ClassData
import com.android.build.api.instrumentation.InstrumentationParameters
import org.objectweb.asm.ClassVisitor
import org.objectweb.asm.MethodVisitor
import org.objectweb.asm.Opcodes

abstract class LogTransform : AsmClassVisitorFactory<InstrumentationParameters.None> {
    override fun createClassVisitor(
        classContext: ClassContext,
        nextClassVisitor: ClassVisitor
    ): ClassVisitor {
      	// 返回一个 ClassVisitor 对象，其内部实现了我们修改 class 文件的逻辑
        return object : ClassVisitor(Opcodes.ASM5, nextClassVisitor) {
            val className = classContext.currentClassData.className
          	// 这里，由于只需要修改方法，故而只重载了 visitMethod 找个方法
            override fun visitMethod(
                access: Int,
                name: String?,
                descriptor: String?,
                signature: String?,
                exceptions: Array<out String>?
            ): MethodVisitor {
                val oldMethodVisitor =
                    super.visitMethod(access, name, descriptor, signature, exceptions)
                // 返回一个 MethodVisitor 对象，其内部实现了我们修改方法的逻辑
                return LogMethodVisitor(className, oldMethodVisitor, access, name, descriptor)
            }
        }
    }

    override fun isInstrumentable(classData: ClassData): Boolean {
        return true
    }
}
```


### 3.2 实现 MethodVisitor

```kotlin
package me.hjhl.gradle.plugin.log

import org.objectweb.asm.MethodVisitor
import org.objectweb.asm.Opcodes
import org.objectweb.asm.commons.AdviceAdapter

class LogMethodVisitor(
    private val className: String,
    nextMethodVisitor: MethodVisitor,
    access: Int,
    name: String?,
    descriptor: String?,
) : AdviceAdapter(Opcodes.ASM5, nextMethodVisitor, access, name, descriptor) {
    override fun onMethodEnter() {
      	// 往栈上加载两个变量，用于后面的函数调用
        mv.visitLdcInsn("LogMethodVisitor")
        mv.visitLdcInsn("enter: $className.$name")
        // 调用 android.util.Log 函数
        mv.visitMethodInsn(
            Opcodes.INVOKESTATIC,
            "android/util/Log",
            "i",
            "(Ljava/lang/String;Ljava/lang/String;)I",
            false
        )
        super.onMethodEnter()
    }

    override fun onMethodExit(opcode: Int) {
        super.onMethodExit(opcode)
    }
}
```


注意到，我们并没有直接继承并实现抽象类 `MethodVisitor`，而是继承 `AdviceAdapter` —— 它是继承自 `MethodVisitor` 的，这样的好处是简化了代码，只需要添加我们需要的逻辑即可 —— 这里我们打印了所调用方法的类及名字。

### 3.3 注册 Transform

原来 Transform API 是通过 `AppExtension` 注册的，现在 AGP 中是通过 `AndroidComponentsExtension` 注册 Transform。用法如下：

```kotlin
+import com.android.build.api.instrumentation.FramesComputationMode
+import com.android.build.api.instrumentation.InstrumentationScope
+import com.android.build.api.variant.AndroidComponentsExtension
+import me.hjhl.gradle.plugin.log.LogTransform
 import org.gradle.api.Plugin
 import org.gradle.api.Project

@@ -9,6 +13,15 @@ class LogPlugin : Plugin<Project> {
     override fun apply(target: Project) {
         log("======== start apply ========")
         log("apply target: ${target.displayName}")
+        val androidComponentsExtension =
+            target.extensions.getByType(AndroidComponentsExtension::class.java)
+        androidComponentsExtension.onVariants { variant ->
+            log("variant: ${variant.name}")
+            variant.instrumentation.apply {
+                transformClassesWith(LogTransform::class.java, InstrumentationScope.PROJECT) {}
+                setAsmFramesComputationMode(FramesComputationMode.COMPUTE_FRAMES_FOR_INSTRUMENTED_METHODS)
+            }
+        }
         log("========  end apply  ========")
     }
```



### 3.4 小结

注意新 Transform API 的注册方式 `AndroidComponentsExtension` 和关键类 `AsmClassVisitorFactory`。至于它们的用法，可以查看本文提供的参考资料或搜索 ASM 相关资料了解。

## 4. 结果

安装并运行 App，看到如下 Log 表示插桩成功。

![](https://static001.geekbang.org/infoq/9e/9e8bf459331cc090a4c5bd3212dd13c5.png)

> Tips：如果对插桩结果有疑问，或者想从字节码角度分析，可以使用 Bytecode Viewer 查看字节码。例如：[https://github.com/Konloch/bytecode-viewer](https://github.com/Konloch/bytecode-viewer "https://github.com/Konloch/bytecode-viewer")

## 5. Q & A

Q：为什么说使用 `buildSrc` 写插件会有坑？

A：插件工程（本文中的 `build-logic/convention`）中，需要依赖 AGP 以访问注册 Transform 的 API，但如果宿主模块中使用了 plugins 语句块的方式引入 AGP 插件，并且使用 `buildSrc` 来编写插件，则会导致 sync/编译报错。

Q：为什么在 `build-logic/convention/build.gradle.kts` 中，要 `exclude org.ow2.asm`？

A：在 AGP 和 Kotlin 中，也使用到了 ASM。如果不屏蔽掉，使用时选错，会导致编译出现**奇怪的报错**。

> Tips：使用时尤为注意，ASM 相关的类是否来自手动依赖的 `org.ow2.asm` 包中。

## 四、总结

如果使用过旧版 Transform API，会发现 TransformAction 的方式节省了非常多的模版代码，比如处理增量编译问题。这使得插件开发者能更专注于核心逻辑实现，提高效率。

## 源码仓库

Github：[HJHL/AsmTourism: ASM tourism on Android (github.com)](https://xie.infoq.cn/link?target=https%3A%2F%2Fgithub.com%2FHJHL%2FAsmTourism "https://xie.infoq.cn/link?target=https%3A%2F%2Fgithub.com%2FHJHL%2FAsmTourism")

## 五、参考资料

- AGP 7.0 release note：[https://developer.android.com/studio/releases/gradle-plugin#7-0-0](https://developer.android.com/studio/releases/gradle-plugin#7-0-0 "https://developer.android.com/studio/releases/gradle-plugin#7-0-0")
    
- AGP API 指南：[https://developer.android.com/reference/tools/gradle-api/7.2/classes](https://developer.android.com/reference/tools/gradle-api/7.2/classes "https://developer.android.com/reference/tools/gradle-api/7.2/classes")
    
- 旧版 Transform API：[https://developer.android.com/reference/tools/gradle-api/7.2/com/android/build/api/transform/Transform](https://developer.android.com/reference/tools/gradle-api/7.2/com/android/build/api/transform/Transform "https://developer.android.com/reference/tools/gradle-api/7.2/com/android/build/api/transform/Transform")
    
- Gradle 项目的目录文件结构说明：[https://docs.gradle.org/current/userguide/organizing_gradle_projects.html](https://docs.gradle.org/current/userguide/organizing_gradle_projects.html "https://docs.gradle.org/current/userguide/organizing_gradle_projects.html")
    
- Gradle Plugin 开发教程：[https://docs.gradle.org/current/userguide/custom_plugins.html](https://docs.gradle.org/current/userguide/custom_plugins.html "https://docs.gradle.org/current/userguide/custom_plugins.html")
    
- Gradle 官方的 Transform 教程 —— 基于 TransformAction API：[https://docs.gradle.org/current/userguide/artifact_transforms.html](https://docs.gradle.org/current/userguide/artifact_transforms.html "https://docs.gradle.org/current/userguide/artifact_transforms.html")
    
- ASM 官网：[https://asm.ow2.io/](https://asm.ow2.io/ "https://asm.ow2.io/")
    
- Java 方法签名：[https://docs.oracle.com/en/java/javase/18/docs/specs/jni/types.html#type-signatures](https://docs.oracle.com/en/java/javase/18/docs/specs/jni/types.html#type-signatures "https://docs.oracle.com/en/java/javase/18/docs/specs/jni/types.html#type-signatures") 

# 原文链接 

[手把手带你实战 AGP 7.x ASM 字节码插桩](https://xie.infoq.cn/article/46ccb9380c6e876e84b204ecc) 