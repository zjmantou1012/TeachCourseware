---
author: zjmantou
title: Android必备：ASM字节码操作
time: 2024-08-29 周四
tags:
  - Android
  - ASM
---
在Android开发中，ASM是一个非常重要的概念。它提供了一种在运行时（Runtime）修改已有类或动态生成新类的方式，为开发者提供了更多的灵活性和控制权。

## [](https://rousetime.com/2023/05/27/Android%E5%BF%85%E5%A4%87%EF%BC%9AASM%E5%AD%97%E8%8A%82%E7%A0%81%E6%93%8D%E4%BD%9C/#%E4%BB%80%E4%B9%88%E6%98%AFASM%EF%BC%9F "什么是ASM？")什么是ASM？

ASM全称为“Java字节码操作框架（Java Bytecode Manipulation Framework）”，它是一个用于生成和转换Java字节码的框架。它可以让你在运行时动态地生成、修改和转换Java字节码，可以做到诸如在类加载时动态修改字节码，或者在执行过程中动态生成新的类等等。

ASM是一个非常轻量级的框架，它的体积不到100KB，但是却具有非常强大的功能。它不仅可以生成类文件，还可以修改已有的类文件，而且这些操作都是在内存中进行的，不需要写入文件。

## [](https://rousetime.com/2023/05/27/Android%E5%BF%85%E5%A4%87%EF%BC%9AASM%E5%AD%97%E8%8A%82%E7%A0%81%E6%93%8D%E4%BD%9C/#ASM%E4%B8%8E%E5%8F%8D%E5%B0%84%E7%9A%84%E5%8C%BA%E5%88%AB "ASM与反射的区别")ASM与反射的区别

在Java中，反射是一个非常常用的功能，它可以让我们在运行时获取类的信息，并且动态地调用类的方法和属性。虽然ASM和反射都是在运行时进行操作，但是它们之间还是有很大的区别的。

首先，反射是在已有的类上进行操作，而ASM可以动态生成新的类或者修改已有的类。这就意味着ASM在某些情况下可以比反射更加灵活和强大。

其次，ASM操作的是字节码，而反射操作的是类的元数据。这就意味着ASM可以做到一些反射无法做到的事情，比如修改类的继承关系、修改类的访问权限等等。

## [](https://rousetime.com/2023/05/27/Android%E5%BF%85%E5%A4%87%EF%BC%9AASM%E5%AD%97%E8%8A%82%E7%A0%81%E6%93%8D%E4%BD%9C/#ASM%E7%9A%84%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF "ASM的使用场景")ASM的使用场景

ASM可以用在很多场景中，比如：

- 代码注入
- AOP（面向切面编程）
- 动态代理
- 字节码加密和混淆
- 动态生成类和方法

除此之外，ASM还可以用来做性能优化。通过对字节码的修改，我们可以让程序在执行时更加高效。比如可以将循环展开、将方法内联、将常量提取等等。

## [](https://rousetime.com/2023/05/27/Android%E5%BF%85%E5%A4%87%EF%BC%9AASM%E5%AD%97%E8%8A%82%E7%A0%81%E6%93%8D%E4%BD%9C/#ASM%E7%9A%84%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5 "ASM的基本概念")ASM的基本概念

在使用ASM时，有一些基本概念需要了解：

### [](https://rousetime.com/2023/05/27/Android%E5%BF%85%E5%A4%87%EF%BC%9AASM%E5%AD%97%E8%8A%82%E7%A0%81%E6%93%8D%E4%BD%9C/#ClassVisitor "ClassVisitor")ClassVisitor

ClassVisitor是ASM中的一个核心接口，它用于访问类的结构。我们可以通过实现ClassVisitor来修改类的结构。

### [](https://rousetime.com/2023/05/27/Android%E5%BF%85%E5%A4%87%EF%BC%9AASM%E5%AD%97%E8%8A%82%E7%A0%81%E6%93%8D%E4%BD%9C/#MethodVisitor "MethodVisitor")MethodVisitor

MethodVisitor是ClassVisitor的子接口，它用于访问方法的结构。我们可以通过实现MethodVisitor来修改方法的结构。

### [](https://rousetime.com/2023/05/27/Android%E5%BF%85%E5%A4%87%EF%BC%9AASM%E5%AD%97%E8%8A%82%E7%A0%81%E6%93%8D%E4%BD%9C/#FieldVisitor "FieldVisitor")FieldVisitor

FieldVisitor是ClassVisitor的子接口，它用于访问字段的结构。我们可以通过实现FieldVisitor来修改字段的结构。

### [](https://rousetime.com/2023/05/27/Android%E5%BF%85%E5%A4%87%EF%BC%9AASM%E5%AD%97%E8%8A%82%E7%A0%81%E6%93%8D%E4%BD%9C/#Type "Type")Type

Type是ASM中的一个核心类，它用于表示Java中的类型。我们可以通过Type来获取一个类的信息，比如类的名称、类的方法、类的字段等等。

### [](https://rousetime.com/2023/05/27/Android%E5%BF%85%E5%A4%87%EF%BC%9AASM%E5%AD%97%E8%8A%82%E7%A0%81%E6%93%8D%E4%BD%9C/#ASMifier "ASMifier")ASMifier

ASMifier是ASM中的一个工具类，它可以将一个已有的类转换成ASM的代码。这个工具类非常有用，可以帮助我们理解ASM的使用。

## [](https://rousetime.com/2023/05/27/Android%E5%BF%85%E5%A4%87%EF%BC%9AASM%E5%AD%97%E8%8A%82%E7%A0%81%E6%93%8D%E4%BD%9C/#ASM%E7%9A%84%E5%AE%9E%E4%BE%8B "ASM的实例")ASM的实例

让我们通过一些实例来了解ASM的使用。

### [](https://rousetime.com/2023/05/27/Android%E5%BF%85%E5%A4%87%EF%BC%9AASM%E5%AD%97%E8%8A%82%E7%A0%81%E6%93%8D%E4%BD%9C/#1-%E4%BB%A3%E7%A0%81%E6%B3%A8%E5%85%A5 "1. 代码注入")1. 代码注入

使用ASM进行代码注入，可以在运行时动态地向已有的方法中加入一些新的代码。比如在方法的开头加入一些日志输出、在方法的结尾加入一些统计代码等等。

下面是一个简单的代码注入的例子：

```java
public class MyClassVisitor extends ClassVisitor {  
    public MyClassVisitor(ClassWriter classWriter) {  
        super(Opcodes.ASM5, classWriter);  
    }  
  
    @Override  
    public MethodVisitor visitMethod(int access, String name, String descriptor, String signature, String[] exceptions) {  
        MethodVisitor mv = super.visitMethod(access, name, descriptor, signature, exceptions);  
        if ("test".equals(name)) {  
            mv = new MethodVisitor(Opcodes.ASM5, mv) {  
                @Override  
                public void visitCode() {  
                    super.visitCode();  
                    mv.visitFieldInsn(Opcodes.GETSTATIC, "java/lang/System", "out", "Ljava/io/PrintStream;");  
                    mv.visitLdcInsn("Entering method test");  
                    mv.visitMethodInsn(Opcodes.INVOKEVIRTUAL, "java/io/PrintStream", "println", "(Ljava/lang/String;)V", false);  
                }  
  
                @Override  
                public void visitInsn(int opcode) {  
                    if (opcode == Opcodes.RETURN) {  
                        mv.visitFieldInsn(Opcodes.GETSTATIC, "java/lang/System", "out", "Ljava/io/PrintStream;");  
                        mv.visitLdcInsn("Exiting method test");  
                        mv.visitMethodInsn(Opcodes.INVOKEVIRTUAL, "java/io/PrintStream", "println", "(Ljava/lang/String;)V", false);  
                    }  
                    super.visitInsn(opcode);  
                }  
            };  
        }  
        return mv;  
    }  
}
```

在这个例子中，我们实现了一个MyClassVisitor类，继承了ClassVisitor。在visitMethod方法中，我们通过判断方法名是否为“test”，来获取方法的MethodVisitor。在MethodVisitor中，我们在方法的开头加入了一条输出“Entering method test”的代码，在方法的结尾加入了一条输出“Exiting method test”的代码。这样每次调用test方法时，都会输出这两条日志信息。

### [](https://rousetime.com/2023/05/27/Android%E5%BF%85%E5%A4%87%EF%BC%9AASM%E5%AD%97%E8%8A%82%E7%A0%81%E6%93%8D%E4%BD%9C/#2-AOP%EF%BC%88%E9%9D%A2%E5%90%91%E5%88%87%E9%9D%A2%E7%BC%96%E7%A8%8B%EF%BC%89 "2. AOP（面向切面编程）")2. AOP（面向切面编程）

AOP是一种编程思想，它可以将一些横切性的功能（比如日志、权限、事务等）统一地应用到多个方法中。使用ASM进行AOP编程，可以在运行时动态地将这些横切性的功能加入到多个方法中。

下面是一个简单的AOP的例子：

```Java
public class MyClassVisitor extends ClassVisitor {  
    public MyClassVisitor(ClassWriter classWriter) {  
        super(Opcodes.ASM5, classWriter);  
    }  
  
    @Override  
    public MethodVisitor visitMethod(int access, String name, String descriptor, String signature, String[] exceptions) {  
        MethodVisitor mv = super.visitMethod(access, name, descriptor, signature, exceptions);  
        if ("test".equals(name)) {  
            mv = new MethodVisitor(Opcodes.ASM5, mv) {  
                @Override  
                public void visitCode() {  
                    mv.visitMethodInsn(Opcodes.INVOKESTATIC, "com/example/LogInterceptor", "before", "()V", false);  
                    super.visitCode();  
                }  
  
                @Override  
                public void visitInsn(int opcode) {  
                    super.visitInsn(opcode);  
                    if (opcode == Opcodes.RETURN) {  
                        mv.visitMethodInsn(Opcodes.INVOKESTATIC, "com/example/LogInterceptor", "after", "()V", false);  
                    }  
                }  
            };  
        }  
        return mv;  
    }  
}
```

在这个例子中，我们实现了一个MyClassVisitor类，继承了ClassVisitor。在visitMethod方法中，我们通过判断方法名是否为“test”，来获取方法的MethodVisitor。在MethodVisitor中，我们在方法的开头加入了一条调用“com/example/LogInterceptor”的before方法的代码，在方法的结尾加入了一条调用“com/example/LogInterceptor”的after方法的代码。这样每次调用test方法时，都会先调用before方法，然后调用test方法，最后调用after方法。

### [](https://rousetime.com/2023/05/27/Android%E5%BF%85%E5%A4%87%EF%BC%9AASM%E5%AD%97%E8%8A%82%E7%A0%81%E6%93%8D%E4%BD%9C/#3-%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86 "3. 动态代理")3. 动态代理

动态代理是一种常用的设计模式，它可以让我们在运行时动态地生成一个代理对象。使用ASM进行动态代理，可以在运行时动态地生成一个实现了指定接口的代理对象。

下面是一个简单的动态代理的例子：

```Java
public class MyClassVisitor extends ClassVisitor {  
    private String className;  
    private String interfaceName;  
  
    public MyClassVisitor(ClassWriter classWriter, String className, String interfaceName) {  
        super(Opcodes.ASM5, classWriter);  
        this.className = className;  
        this.interfaceName = interfaceName;  
    }  
  
    @Override  
    public void visit(int version, int access, String name, String signature, String superName, String[] interfaces) {  
        super.visit(version, access, className, signature, "java/lang/Object", new String[]{interfaceName});  
    }  
  
    @Override  
    public MethodVisitor visitMethod(int access, String name, String descriptor, String signature, String[] exceptions) {  
        MethodVisitor mv = super.visitMethod(access, name, descriptor, signature, exceptions);  
        if ("invoke".equals(name)) {  
            mv = new MethodVisitor(Opcodes.ASM5, mv) {  
                @Override  
                public void visitCode() {  
                    super.visitCode();  
                    mv.visitVarInsn(Opcodes.ALOAD, 0);  
                    mv.visitFieldInsn(Opcodes.GETFIELD, className, "handler", "Ljava/lang/Object;");  
                    mv.visitTypeInsn(Opcodes.CHECKCAST, interfaceName);  
                    mv.visitVarInsn(Opcodes.ALOAD, 1);  
                    mv.visitVarInsn(Opcodes.ALOAD, 2);  
                    mv.visitMethodInsn(Opcodes.INVOKEINTERFACE, interfaceName, "invoke", "(Ljava/lang/Object;Ljava/lang/reflect/Method;[Ljava/lang/Object;)Ljava/lang/Object;", true);  
                    mv.visitInsn(Opcodes.ARETURN);  
                }  
            };  
        }  
        return mv;  
    }  
}
```

在这个例子中，我们实现了一个MyClassVisitor类，继承了ClassVisitor。在visit方法中，我们将类的名称修改为指定的名称，将类实现的接口修改为指定的接口。在visitMethod方法中，我们通过判断方法名是否为“invoke”，来获取方法的MethodVisitor

### [](https://rousetime.com/2023/05/27/Android%E5%BF%85%E5%A4%87%EF%BC%9AASM%E5%AD%97%E8%8A%82%E7%A0%81%E6%93%8D%E4%BD%9C/#%E4%BD%BF%E7%94%A8ASM%E8%BF%9B%E8%A1%8C%E5%AD%97%E8%8A%82%E7%A0%81%E5%8A%A0%E5%AF%86%E5%92%8C%E6%B7%B7%E6%B7%86 "使用ASM进行字节码加密和混淆")使用ASM进行字节码加密和混淆

使用ASM可以对字节码进行加密和混淆，增强代码的安全性。可以通过修改常量池、修改方法名和字段名等方式来达到加密和混淆的效果。

下面是一个简单的字节码加密和混淆的例子：

```Java
public class MyClassVisitor extends ClassVisitor {  
    private String key;  
  
    public MyClassVisitor(ClassWriter classWriter, String key) {  
        super(Opcodes.ASM5, classWriter);  
        this.key = key;  
    }  
  
    @Override  
    public FieldVisitor visitField(int access, String name, String descriptor, String signature, Object value) {  
        if ((access & (Opcodes.ACC_PUBLIC | Opcodes.ACC_PROTECTED)) != 0) {  
            return super.visitField(access, name, descriptor, signature, value);  
        } else {  
            FieldVisitor fv = super.visitField(access, name + "_" + key, descriptor, signature, value);  
            fv.visitAnnotation("Lcom/example/Encrypted;", true);  
            return fv;  
        }  
    }  
  
    @Override  
    public MethodVisitor visitMethod(int access, String name, String descriptor, String signature, String[] exceptions) {  
        if ((access & (Opcodes.ACC_PUBLIC | Opcodes.ACC_PROTECTED)) != 0) {  
            return super.visitMethod(access, name, descriptor, signature, exceptions);  
        } else {  
            MethodVisitor mv = super.visitMethod(access, name + "_" + key, descriptor, signature, exceptions);  
            mv.visitAnnotation("Lcom/example/Obfuscated;", true);  
            return mv;  
        }  
    }  
  
    @Override  
    public void visitEnd() {  
        AnnotationVisitor av = super.visitAnnotation("Lcom/example/EncryptedClass;", true);  
        av.visit("key", key);  
        super.visitEnd();  
    }  
}
```

在这个例子中，我们实现了一个MyClassVisitor类，继承了ClassVisitor。在visitField方法中，我们判断字段是否为公有或者受保护的，如果是，则不做处理；如果不是，则将字段名加上指定的key，并且加上注解“@Encrypted”。在visitMethod方法中，我们同样地判断方法是否为公有或者受保护的，如果是，则不做处理；如果不是，则将方法名加上指定的key，并且加上注解“@Obfuscated”。在visitEnd方法中，我们加上注解“@EncryptedClass”，并且添加一个属性“key”，代表加密的key。

### [](https://rousetime.com/2023/05/27/Android%E5%BF%85%E5%A4%87%EF%BC%9AASM%E5%AD%97%E8%8A%82%E7%A0%81%E6%93%8D%E4%BD%9C/#%E4%BD%BF%E7%94%A8ASM%E5%8A%A8%E6%80%81%E7%94%9F%E6%88%90%E7%B1%BB%E5%92%8C%E6%96%B9%E6%B3%95 "使用ASM动态生成类和方法")使用ASM动态生成类和方法

使用ASM可以动态生成类和方法，这是动态代理、AOP等功能的基础。可以使用ASM生成的类，继承已有的类或者实现指定的接口，具备与原类相似的功能。

下面是一个简单的动态生成类和方法的例子：

```Java
public class MyClassWriter extends ClassWriter {  
    public MyClassWriter() {  
        super(ClassWriter.COMPUTE_MAXS);  
    }  
  
    public void generateClass() {  
        visit(Opcodes.V1_8, Opcodes.ACC_PUBLIC, "com/example/DynamicClass", null, "java/lang/Object", null);  
  
        // 添加字段    
visitField(Opcodes.ACC_PRIVATE, "name", "Ljava/lang/String;", null, null).visitEnd();  
  
        // 添加构造方法    
MethodVisitor constructorVisitor = visitMethod(Opcodes.ACC_PUBLIC, "<init>", "()V", null, null);  
        constructorVisitor.visitCode();  
        constructorVisitor.visitVarInsn(Opcodes.ALOAD, 0);  
        constructorVisitor.visitMethodInsn(Opcodes.INVOKESPECIAL, "java/lang/Object", "<init>", "()V", false);  
        constructorVisitor.visitInsn(Opcodes.RETURN);  
        constructorVisitor.visitMaxs(0, 0);  
        constructorVisitor.visitEnd();  
  
        // 添加setName方法    
MethodVisitor setNameVisitor = visitMethod(Opcodes.ACC_PUBLIC, "setName", "(Ljava/lang/String;)V", null, null);  
        setNameVisitor.visitCode();  
        setNameVisitor.visitVarInsn(Opcodes.ALOAD, 0);  
        setNameVisitor.visitVarInsn(Opcodes.ALOAD, 1);  
        setNameVisitor.visitFieldInsn(Opcodes.PUTFIELD, "com/example/DynamicClass", "name", "Ljava/lang/String;");  
        setNameVisitor.visitInsn(Opcodes.RETURN);  
        setNameVisitor.visitMaxs(0, 0);  
        setNameVisitor.visitEnd();  
  
        // 添加getName方法    
MethodVisitor getNameVisitor = visitMethod(Opcodes.ACC_PUBLIC, "getName", "()Ljava/lang/String;", null, null);  
        getNameVisitor.visitCode();  
        getNameVisitor.visitVarInsn(Opcodes.ALOAD, 0);  
        getNameVisitor.visitFieldInsn(Opcodes.GETFIELD, "com/example/DynamicClass", "name", "Ljava/lang/String;");  
        getNameVisitor.visitInsn(Opcodes.ARETURN);  
        getNameVisitor.visitMaxs(0, 0);  
        getNameVisitor.visitEnd();  
  
        visitEnd();  
    }  
}
```

在这个例子中，我们实现了一个MyClassWriter类，继承了ClassWriter。在generateClass方法中，我们使用visit方法生成了一个名为“com/example/DynamicClass”的类，继承自Object类。接着，我们添加了一个私有字段“name”，一个公有的构造方法和两个公有的方法“setName”和“getName”。在这两个方法中，我们使用visitFieldInsn等方法，访问了字段“name”的值，并且返回了它。这样，我们就通过ASM动态生成了一个类和其中的方法。

## [](https://rousetime.com/2023/05/27/Android%E5%BF%85%E5%A4%87%EF%BC%9AASM%E5%AD%97%E8%8A%82%E7%A0%81%E6%93%8D%E4%BD%9C/#%E6%80%BB%E7%BB%93 "总结")总结

本文介绍了ASM的基本概念、使用场景以及一个简单的实例。虽然ASM的使用需要一定的学习成本，但是它可以为我们提供更加灵活和强大的功能，帮助我们解决一些难题。如果你还没有使用过ASM，我建议你去尝试一下，相信你会有新的发现

## [](https://rousetime.com/2023/05/27/Android%E5%BF%85%E5%A4%87%EF%BC%9AASM%E5%AD%97%E8%8A%82%E7%A0%81%E6%93%8D%E4%BD%9C/#%E6%8E%A8%E8%8D%90 "推荐")推荐

[android_startup](https://github.com/idisfkj/android-startup): 提供一种在应用启动时能够更加简单、高效的方式来初始化组件，优化启动速度。不仅支持Jetpack App Startup的全部功能，还提供额外的同步与异步等待、线程控制与多进程支持等功能。

[AwesomeGithub](https://github.com/idisfkj/AwesomeGithub): 基于Github的客户端，纯练习项目，支持组件化开发，支持账户密码与认证登陆。使用Kotlin语言进行开发，项目架构是基于JetPack&DataBinding的MVVM；项目中使用了Arouter、Retrofit、Coroutine、Glide、Dagger与Hilt等流行开源技术。

[flutter_github](https://github.com/idisfkj/flutter_github): 基于Flutter的跨平台版本Github客户端，与AwesomeGithub相对应。

[android-api-analysis](https://github.com/idisfkj/android-api-analysis): 结合详细的Demo来全面解析Android相关的知识点, 帮助读者能够更快的掌握与理解所阐述的要点。

[daily_algorithm](https://github.com/idisfkj/daily_algorithm): 每日一算法，由浅入深，欢迎加入一起共勉。


[原文链接](https://rousetime.com/2023/05/27/Android%E5%BF%85%E5%A4%87%EF%BC%9AASM%E5%AD%97%E8%8A%82%E7%A0%81%E6%93%8D%E4%BD%9C/#%E4%BD%BF%E7%94%A8ASM%E8%BF%9B%E8%A1%8C%E5%AD%97%E8%8A%82%E7%A0%81%E5%8A%A0%E5%AF%86%E5%92%8C%E6%B7%B7%E6%B7%86) 

