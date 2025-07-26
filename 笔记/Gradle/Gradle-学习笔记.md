---
author: zjmantou
title: Gradle-学习笔记
time: 2025-07-26 周六
tags:
  - Gradle
  - ASM
---
# 示例

- gradle版本：8.2.2
- kotlin：1.8.10

使用DefaullTask实现ASM插庄，拦截onClick方法

- 使用build-logic库模式，参照nowinandroid项目
- asm库：
	- org.ow2.asm:asm:9.7
	- org.ow2.asm:asm-commons:9.7

## 注册任务

使用TaskProvider

```kotlin
class OnclickPlugins : Plugin<Project> {  
    override fun apply(target: Project) {  
       target.plugins.withType(AppPlugin::class.java) {  
          val androidComponents =  
             target.extensions.getByType<ApplicationAndroidComponentsExtension>()  
  
          androidComponents.onVariants { variant ->  
             val taskProvider =  
                target.tasks.register<OnclickTask>("${variant.name}OnclickTask")  
  
             variant.artifacts.forScope(ScopedArtifacts.Scope.PROJECT)  
                .use(taskProvider)  
                .toTransform(  
                   ScopedArtifact.CLASSES,  
                   OnclickTask::allJars,  
                   OnclickTask::allDirectories,  
                   OnclickTask::output  
                )  
          }  
       }    }  
}
```

## OnclickTask

打开JarOutputStream
需要asm操作的file使用ClassVisitor操作，不需要的直接复制到输出文件；

```kotlin
abstract class OnclickTask : DefaultTask() {  
  
    @get:InputFiles  
    abstract val allJars: ListProperty<RegularFile>  
  
    @get:InputFiles  
    abstract val allDirectories: ListProperty<Directory>  
  
    @get:OutputFile  
    abstract val output: RegularFileProperty  
  
    @TaskAction    fun taskAction() {  
       println("OnclickTask")  
       allJars.get().forEach {  
          println("allJars: fileName ${it.asFile.name}")  
       }  
  
       allDirectories.get().forEach { directory ->  
          println("allDirectories: fileName ${directory.asFile.name}")  
          directory.asFile.walk().forEach { file ->  
             if (file.isFile) {  
                println("file: fileName ${file.name}")  
             }          }  
       }       println("output: fileName ${output.get().asFile.name}")  
  
       val outputJarFile = output.get().asFile  
       // 确保输出目录存在  
       JarOutputStream(outputJarFile.outputStream()).use { jos ->  
          // 处理所有输入的 JAR 包  
          allJars.get().forEach { regularFile ->  
             val jarFile = JarFile(regularFile.asFile)  
             jarFile.entries().asSequence().forEach { entry ->  
                val entryName = entry.name  
                println("allJars entry: name: $entryName")  
                if (!entry.isDirectory && entryName.endsWith(".class")) {  
                   // 如果是 .class 文件，进行字节码转换  
                   val inputStream = jarFile.getInputStream(entry)  
                   val transformedBytes = transformClass(inputStream.readBytes())  
                   // 将转换后的字节码写入输出 JAR                   val newEntry = JarEntry(entryName)  
                   jos.putNextEntry(newEntry)  
                   jos.write(transformedBytes)  
                   jos.closeEntry()  
                } else {  
                   // 非 .class 文件直接复制  
                   val inputStream = jarFile.getInputStream(entry)  
                   val newEntry = JarEntry(entryName)  
                   jos.putNextEntry(newEntry)  
                   inputStream.copyTo(jos)  
                   jos.closeEntry()  
                }             }  
             jarFile.close()  
          }  
  
          // 处理所有输入的类目录  
          allDirectories.get().forEach { directory ->  
             val baseDirPath = directory.asFile.toPath()  
             println("baseDirPath: $baseDirPath")  
             Files.walk(baseDirPath).use { paths ->  
                paths.forEach { path ->  
                   println("allDirectories path: ${path.toString()}")  
                   if (Files.isRegularFile(path) && path.toString().endsWith(".class")) {  
                      val relativePath = baseDirPath.relativize(path)  
                      println("relativePath = $relativePath")  
                      val entryName = relativePath.toString().replace(File.separatorChar, '/')  
                      println("entryName = $entryName")  
                      val bytes = Files.readAllBytes(path)  
                      val transformedBytes = transformClass(bytes)  
  
                      val newEntry = JarEntry(entryName)  
                      jos.putNextEntry(newEntry)  
                      jos.write(transformedBytes)  
                      jos.closeEntry()  
                   } else if (Files.isDirectory(path) && path != baseDirPath) {  
                      val relativePath = baseDirPath.relativize(path)  
                      println("relativePath = $relativePath")  
                      val entryName = relativePath.toString().replace(File.separatorChar, '/') + "/"  
                      println("entryName = $entryName")  
                      val newEntry = JarEntry(entryName)  
                      jos.putNextEntry(newEntry)  
                      jos.closeEntry()  
                   }                }  
             }          }       }       println("ASM transformation completed. Output: ${outputJarFile.absolutePath}")  
    }  
    private fun transformClass(classBytes: ByteArray): ByteArray {  
       val classReader = ClassReader(classBytes)  
       // COMPUTE_MAXS 自动计算栈帧大小  
       val classWriter =  
          ClassWriter(classReader, ClassWriter.COMPUTE_MAXS)  
       val classVisitor = OnClickListenerClassVisitor(classWriter)  
       // EXPAND_FRAMES 确保正确的栈帧信息  
       classReader.accept(classVisitor, ClassReader.EXPAND_FRAMES)  
       return classWriter.toByteArray()  
    }}
```

## ClassVisitor

进行条件判断，符合条件的返回自定义的MethodVisitor

```kotlin
class OnClickListenerClassVisitor(  
    api: Int,  
    cv: ClassVisitor?  
) : ClassVisitor(api, cv) {  
  
    constructor(cv: ClassVisitor?) : this(Opcodes.ASM9, cv)  
  
    private var className: String = ""  
    private var classSuperName: String = ""  
    private var classInterfaces: Array<out String>? = null  
  
    override fun visit(  
       version: Int,  
       access: Int,  
       name: String,  
       signature: String?,  
       superName: String?,  
       interfaces: Array<out String>?  
    ) {  
       this.className = name  
       classSuperName = superName.orEmpty()  
       classInterfaces = interfaces  
       println("visit name : $name, signature: $signature, superName: $superName")  
       super.visit(version, access, name, signature, superName, interfaces)  
    }  
    override fun visitMethod(  
       access: Int,  
       name: String?,  
       descriptor: String?,  
       signature: String?,  
       exceptions: Array<out String?>?  
    ): MethodVisitor? {  
       println("visitMethod : name:$name, descriptor:$descriptor, signature:$signature, exceptions:$exceptions")  
       val mv = super.visitMethod(access, name, descriptor, signature, exceptions)  
       // 拦截 View.OnClickListener 接口的 onClick 方法  
       // 接口的方法通常是 public abstract       // name: onClick       // descriptor: (Landroid/view/View;)V (表示输入一个 View 对象，返回 void)       // superName 是接口的名称 (android/view/View$OnClickListener)       // 检查当前类是否实现了 OnClickListener 接口  
       // 这里只是一个简单的检查，更严谨的应该通过 ClassReader 检查接口列表  
       // 或者更准确地检查方法签名是否匹配 `android/view/View$OnClickListener.onClick`       // 为了演示，我们假设我们正在处理一个实现 OnClickListener 的类，  
       // 并且我们只想修改名为 'onClick' 且描述符匹配的方法。  
       // 一个更 robust 的方法是，在 visitClass 中记录实现的接口，然后在这里判断  
       val isOnClickListener = className.contains("OnClickListener") ||  
             classInterfaces?.any { it == "android/view/View\$OnClickListener" } == true  
       if (isOnClickListener && name == "onClick" && descriptor == "(Landroid/view/View;)V") {  
          println("Found onClick method in class: $className")  
          return OnClickMethodVisitor(api, mv, className)  
       }       println("Not Found onClick method in class: $className !!!")  
       return mv  
    }  
  
}
```

## MethodVisitor

对方法操作字节码

```kotlin
// MethodVisitor 负责遍历方法内部的指令  
class OnClickMethodVisitor(api: Int, mv: MethodVisitor?, private val className: String) :  
    MethodVisitor(api, mv) {  
    override fun visitCode() {  
       super.visitCode()  
       // 在方法开始时插入日志  
       // GETSTATIC System.out Ljava/io/PrintStream;  
       mv?.visitFieldInsn(Opcodes.GETSTATIC, "java/lang/System", "out", "Ljava/io/PrintStream;")  
       // LDC "onClick method start: " + className  
       mv?.visitLdcInsn("ASM: onClick method start in $className")  
       // INVOKEVIRTUAL PrintStream.println (Ljava/lang/String;)V  
       mv?.visitMethodInsn(  
          Opcodes.INVOKEVIRTUAL,  
          "java/io/PrintStream",  
          "println",  
          "(Ljava/lang/String;)V",  
          false  
       )  
    }  
    override fun visitInsn(opcode: Int) {  
       // 在方法返回前插入日志  
       // ASM 中的 RETURN 指令包括: RETURN, ARETURN, IRETURN, LRETURN, FRETURN, DRETURN  
       if (opcode == Opcodes.RETURN || opcode == Opcodes.ARETURN ||  
          opcode == Opcodes.IRETURN || opcode == Opcodes.LRETURN ||  
          opcode == Opcodes.FRETURN || opcode == Opcodes.DRETURN  
       ) {  
          // GETSTATIC System.out Ljava/io/PrintStream;  
          mv?.visitFieldInsn(  
             Opcodes.GETSTATIC,  
             "java/lang/System",  
             "out",  
             "Ljava/io/PrintStream;"  
          )  
          // LDC "onClick method end: " + className  
          mv?.visitLdcInsn("ASM: onClick method end in $className")  
          // INVOKEVIRTUAL PrintStream.println (Ljava/lang/String;)V  
          mv?.visitMethodInsn(  
             Opcodes.INVOKEVIRTUAL,  
             "java/io/PrintStream",  
             "println",  
             "(Ljava/lang/String;)V",  
             false  
          )  
       }       super.visitInsn(opcode)  
    }}
```

# 学习路径

Systematically learning Gradle和ASM字节码操作确实需要一些时间和精力，但投资绝对值得，特别是对于 Android 开发中的高级构建优化和插件开发。

---

## 学习 Gradle

Gradle 是一个功能强大且灵活的构建自动化工具。对于 Android 开发者来说，理解它尤为重要，因为 Android 构建系统就是基于 Gradle 的。

### 官方文档和教程

1. **Gradle 官方用户手册 (Gradle Official User Manual)**:
    
    - **优点**: 最权威、最全面的资料。它详细解释了 Gradle 的核心概念、DSL (Domain Specific Language)、任务、插件、依赖管理、多项目构建等。
        
    - **中文版**: 虽然有时更新不如英文版及时，但也有中文翻译版本可供参考：[Gradle用户指南官方文档中文版](https://doc.yonyoucloud.com/doc/wiki/project/GradleUserGuide-Wiki/index.html)。
        
    - **建议**: 这是你学习 Gradle 的首选。从 "入门教程" 开始，逐步深入各个章节。
        
2. **Android Developers 官方文档 (Gradle Build Overview)**:
    
    - **优点**: 针对 Android 项目，解释了 Gradle 如何与 Android Gradle Plugin (AGP) 协同工作，以及 Android 特有的构建概念（如 Build Types, Product Flavors, Variants, Artifacts API 等）。
        
    - **链接**: [Gradle 构建概览 | Android Studio](https://developer.android.com/build/gradle-build-overview?hl=zh-cn)
        
    - **建议**: 在学习 Gradle 基础后，结合这个文档深入理解 Android 构建流程。
        

### 书籍推荐

1. **《Gradle in Action》 by Benjamin Muschko**:
    
    - **优点**: 这本书被广泛认为是学习 Gradle 的最佳书籍之一。它涵盖了 Gradle 的各个方面，从基础到高级主题，并提供了大量的实际示例。虽然是英文书，但内容非常深入和实用。
        
    - **建议**: 如果你能接受英文原版书籍，强烈推荐这本书，它能帮助你建立坚实的 Gradle 知识体系。
        
2. **《Gradle for Android》 by Kevin Pelgrims 或 Ken Kousen**:
    
    - **优点**: 专门针对 Android 开发者，更侧重于 Android 项目中 Gradle 的应用，例如如何定制构建、处理依赖、使用 Product Flavors 等。
        
    - **建议**: 适合希望快速掌握 Gradle 在 Android 开发中实践的开发者。
        

### 实践项目

- **从零开始构建小型项目**: 尝试不依赖 Android Studio 的自动配置，手动创建 `settings.gradle.kts` 和 `build.gradle.kts` 文件，逐步添加任务和依赖。
    
- **查看开源项目的 Gradle 配置**: 学习大型开源项目（如 Jetpack Compose 库、知名开源应用）是如何组织和配置其 Gradle 构建的。
    

---

## 学习 ASM 字节码操作

ASM 是一个强大的 Java 字节码操作框架，它允许你在运行时或编译时直接修改或生成 `.class` 文件。

### 官方文档和示例

1. **ASM 官方网站**:
    
    - **链接**: [ASM Official Website](https://asm.ow2.io/)
        
    - **优点**: 包含了用户指南 (User Guide)、Javadoc 和 FAQ。用户指南是学习 ASM 概念和 API 的主要资源。
        
    - **建议**: 阅读其官方用户指南是入门 ASM 的必经之路。它会介绍 ClassReader, ClassVisitor, MethodVisitor, FieldVisitor 等核心组件，以及如何使用它们进行字节码操作。
        

### 博客、文章和教程

ASM 相对底层和复杂，很多高质量的学习资料以博客文章和在线教程的形式存在：

1. **Medium 系列文章**: 在 Medium 上搜索 "Java bytecode manipulation" 或 "ASM bytecode" 会找到大量有用的文章，很多作者会通过具体的例子来解释 ASM 的使用。
    
    - 例如："[How to manipulate byte-code with ASM](https://medium.com/@hugosafilho/how-to-manipulate-byte-code-with-asm-3c9b8fbe0f8c)" 或 "[JVM Bytecode Manipulation and Instrumentation](https://medium.com/@AlexanderObregon/jvm-bytecode-manipulation-and-instrumentation-0ae7b05fc267)"。
        
2. **GitHub 上的示例项目**: 许多开发者会在 GitHub 上分享 ASM 相关的代码示例或小型项目。搜索 "ASM Java example" 或 "Android ASM plugin" 可以找到很多有用的参考。
    
3. **美团技术团队等国内大厂的技术博客**: 国内一些大厂在热修复、性能优化、无痕埋点等领域会使用字节码插桩技术，他们的技术博客（如美团技术团队的“字节码增强技术探索”系列）会分享相关经验和原理。
    

### Java 字节码基础知识

在深入 ASM 之前，强烈建议你先了解 Java 字节码的基础知识。这会帮助你理解 ASM API 所操作的每个指令和结构。

- **JVM 规范 (Java Virtual Machine Specification)**: 这是最权威的来源，虽然枯燥但非常准确。你可以重点关注 `.class` 文件格式、常量池、方法表、属性等章节，以及指令集部分。
    
- **在线教程和博客**: 许多博客和网站有关于 Java 字节码的入门介绍，例如 "An Introduction to Java Bytecode" 或 "Java Bytecode Simplified"。这些会解释栈帧、局部变量表、操作数栈、各种指令的含义。
    
- **`javap` 工具**: Java SDK 自带的 `javap` 命令可以将 `.class` 文件反编译成可读的字节码指令，这是理解代码如何被编译成字节码的强大工具。在命令行中使用 `javap -c YourClass.class` 可以查看方法的字节码指令。
    

### 实践项目

- **从小例子开始**: 不要一开始就尝试复杂的插桩。从最简单的例子开始，比如：
    
    - 添加一个 `System.out.println` 语句到方法开头。
        
    - 修改一个方法的返回值。
        
    - 添加一个简单的字段。
        
- **结合 Gradle 插件**: 当你掌握了 ASM 基础后，尝试将 ASM 逻辑整合到 Gradle 插件中，就像我们之前讨论的，利用 `AndroidComponentsExtension` 和 `ScopedArtifacts` 来处理 Android 项目的字节码。
    

---

### 总结建议

1. **先学 Gradle 基础**：熟练掌握 Gradle 的任务、依赖、插件和核心 DSL。
    
2. **深入学习 Android Gradle Plugin**：理解 AGP 如何与 Gradle 协同，特别是构建生命周期和 Artifacts API。
    
3. **学习 Java 字节码基础**：这是理解 ASM 的前提。
    
4. **逐步学习 ASM**：从官方文档开始，结合在线教程和实践，先从简单的修改入手，逐步增加复杂度。
    
5. **动手实践是王道**：理论知识再多，不如亲手写代码调试来得实在。