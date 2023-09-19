---
author: zjmantou
title: （1）JNI开发必学C-C++基础
time: 2023-09-19 周二
tags:
  - Android
  - NDK
  - JNI
---


**JNI开发系列目录**

1. [JNI开发必学C++基础](https://blog.csdn.net/luo_boke/article/details/126908373 "https://blog.csdn.net/luo_boke/article/details/126908373")
2. [JNI开发必学C++使用实践](https://blog.csdn.net/luo_boke/article/details/126920916 "https://blog.csdn.net/luo_boke/article/details/126920916")
3. [Android Studio 4.0.+NDK项目开发详细教学](https://blog.csdn.net/luo_boke/article/details/107306531 "https://blog.csdn.net/luo_boke/article/details/107306531")
4. [Android NDK与JNI的区别有何不同？](https://blog.csdn.net/luo_boke/article/details/107234358 "https://blog.csdn.net/luo_boke/article/details/107234358")
5. [Android Studio 4.0.+NDK .so库生成打包](https://blog.csdn.net/luo_boke/article/details/109362013 "https://blog.csdn.net/luo_boke/article/details/109362013")
6. [Android JNI的深度进阶学习](https://blog.csdn.net/luo_boke/article/details/109455910 "https://blog.csdn.net/luo_boke/article/details/109455910")
7. [Android Studio 4.0.+NDK开发 This files is not part of the project](https://blog.csdn.net/luo_boke/article/details/109318721 "https://blog.csdn.net/luo_boke/article/details/109318721")

---

**`以Android studio 7.0.0来分析讲解，所以是Android最新版NDK项目创建，其截图可能与低版本不一样。`**

---

## 前言

C++ 是一种静态类型的、编译式的、通用的、大小写敏感的、不规则的编程语言，支持过程化编程、面向对象编程和泛型编程。  
C++ 被认为是一种中级语言，它综合了高级语言和低级语言的特点。C++ 是 C 的一个超集，事实上，任何合法的 C 程序都是合法的 C++ 程序。

## 标准库

标准的 C++ 由三个重要部分组成：

1. 核心语言，提供了所有构件块，包括变量、数据类型和常量，等等。
2. C++ 标准库，提供了大量的函数，用于操作文件、字符串等。
3. 标准模板库（STL），提供了大量的方法，用于操作数据结构等。

## C++的使用与特性

大多数的 C++ 编译器并不在乎源文件的扩展名，但是如果您未指定扩展名，则默认使用 .cpp，其它的文件名也可以是.h文件。  
C++ 完全支持面向对象的程序设计，包括面向对象开发的四大特性：**封装、抽象、继承、多态**

**C++ 的使用场景**

1. 基本上每个应用程序领域的程序员都有使用 C++。
2. C++ 通常用于编写设备驱动程序和其他要求实时性的直接操作硬件的软件。
3. C++ 广泛用于教学和研究。

**C++ 基本语法**

1. 对象 - 对象具有状态和行为。例如：一只狗的状态 - 颜色、名称、品种，行为 - 摇动、叫唤、吃。对象是类的实例。
2. 类 - 类可以定义为描述对象行为/状态的模板/蓝图。
3. 方法 - 从基本上说，一个方法表示一种行为。一个类可以包含多个方法。可以在方法中写入逻辑、操作数据以及执行所有的动作。
4. 即时变量 - 每个对象都有其独特的即时变量。对象的状态是由这些即时变量的值创建的。

**定义常量**  
在 C++ 中，有两种简单的定义常量的方式：  
使用 #define 预处理器。  
使用 const 关键字。

**类的继承**  
C++ 类的继承和 Java 也是大同小异，其格式如下：class B: access-specifier A。  
其中 access-specifier 是访问修饰符， 是 public、protected 或 private 其中的一个。访问修饰符的作用如下：

1. 公有继承（public）：当一个类派生自公有基类时，基类的公有成员也是派生类的公有成员，基类的保护成员也是派生类的保护成员，基类的私有成员不能直接被派生类访问，但是可以通过调用基类的公有和保护成员来访问。
2. 保护继承（protected）： 当一个类派生自保护基类时，基类的公有和保护成员将成为派生类的保护成员。
3. 私有继承（private）：当一个类派生自私有基类时，基类的公有和保护成员将成为派生类的私有成员。

注意：`C++支持多继承`，class <派生类名>:<继承方式1><基类名1>

## C++ 关键字

|Column|Column|Column|Column|
|---|---|---|---|
|asm|else|new|this|
|auto|enum|operator|throw|
|bool|explicit|private|true|
|break|export|protected|try|
|case|extern|public|typedef|
|catch|false|register|typeid|
|char|float|reinterpret_cast|typename|
|class|for|return|union|
|const|friend|short|unsigned|
|const_cast|goto|signed|using|
|continue|if|sizeof|virtual|
|default|inline|static|void|
|delete|int|static_cast|volatile|
|do|long|struct|wchar_t|
|double|mutable|switch|while|
|dynamic_cast|namespace|template||

## C++ 基本数据类型

|类型|关键字|描述|
|---|---|---|
|布尔型|bool|存储值 true 或 false。|
|字符型|char|一个字符（八位）的整数类型。|
|整型|int|对机器而言，整数的最自然的大小。|
|浮点型|float|单精度浮点值。单精度是这样的格式，1位符号，8位整数，23位小数。|
|双浮点型|double|双精度浮点值。双精度是1位符号，11位指数，52位小数。|
|无类型|void|表示类型的缺失。|
|宽字符型|wchar_t|宽字符类型。|

## 数据范围

|类型|位|范围|
|---|---|---|
|char|1 个字节|-128 到 127 或者 0 到 255|
|unsigned char|1 个字节|0 到 255|
|signed char|1 个字节|-128 到 127|
|int|4 个字节|-2147483648 到 2147483647|
|unsigned int|4个字节|0 到 4294967295|
|signed int|4 个字节|-2147483648 到 2147483647|
|short int|2 个字节|-32768 到 32767|
|unsigned short int|2 个字节|0 到 65,535|
|signed short int|2 个字节|-32768 到 32767|
|long int|8 个字节|-9,223,372,036,854,775,808 到 9,223,372,036,854,775,807|
|signed long int|8 个字节|-9,223,372,036,854,775,808 到 9,223,372,036,854,775,807|
|unsigned long int|8 个字节|0 到 18,446,744,073,709,551,615|
|float|4 个字节|精度型占4个字节（32位）内存空间，+/- 3.4e +/- 38 (~7 个数字)|
|double|8 个字节|双精度型占8 个字节（64位）内存空间，+/- 1.7e +/- 308 (~15 个数字)|
|long double|16 个字节|长双精度型 16 个字节（128位）内存空间，可提供18-19位有效数字。|
|wchar_t|2 或 4 个字节|1 个宽字符|

signed 和 unsigned 指定了数据是否有正负； short 和 long 主要指定了数据的内存大小。

## 转义符

|转义序列|含义|
|---|---|
|\|\ 字符|
|’|’ 字符|
|"|" 字符|
|?|? 字符|
|\a|警报铃声|
|\b|退格键|
|\f|换页符|
|\n|换行符|
|\r|回车|
|\t|水平制表符|
|\v|垂直制表符|
|\ooo|一到三位的八进制数|
|\xhh . . .|一个或多个数字的十六进制数|

## 总结

[JNI](https://so.csdn.net/so/search?q=JNI&spm=1001.2101.3001.7020 "https://so.csdn.net/so/search?q=JNI&spm=1001.2101.3001.7020")标准作为Java平台的一部分，提供了与编译型语言进行交互的手段，尤其是对C/C++的交互。C/C++是JNI开发必须要掌握的技术，下一篇博文[C++使用实践](https://blog.csdn.net/luo_boke/article/details/126920916 "https://blog.csdn.net/luo_boke/article/details/126920916")来讲解C++的实践使用。

---

**相关链接**：

1. [JNI开发必学C++基础](https://blog.csdn.net/luo_boke/article/details/126908373 "https://blog.csdn.net/luo_boke/article/details/126908373")
2. [JNI开发必学C++使用实践](https://blog.csdn.net/luo_boke/article/details/126920916 "https://blog.csdn.net/luo_boke/article/details/126920916")
3. [Android Studio 4.0.+NDK项目开发详细教学](https://blog.csdn.net/luo_boke/article/details/107306531 "https://blog.csdn.net/luo_boke/article/details/107306531")
4. [Android NDK与JNI的区别有何不同？](https://blog.csdn.net/luo_boke/article/details/107234358 "https://blog.csdn.net/luo_boke/article/details/107234358")
5. [Android Studio 4.0.+NDK .so库生成打包](https://blog.csdn.net/luo_boke/article/details/109362013 "https://blog.csdn.net/luo_boke/article/details/109362013")
6. [Android JNI的深度进阶学习](https://blog.csdn.net/luo_boke/article/details/109455910 "https://blog.csdn.net/luo_boke/article/details/109455910")
7. [Android Studio 4.0.+NDK开发 This files is not part of the project](https://blog.csdn.net/luo_boke/article/details/109318721 "https://blog.csdn.net/luo_boke/article/details/109318721")

**博客书写不易，您的点赞收藏是我前进的动力，千万别忘记点赞、 收藏 ^ _ ^ !**