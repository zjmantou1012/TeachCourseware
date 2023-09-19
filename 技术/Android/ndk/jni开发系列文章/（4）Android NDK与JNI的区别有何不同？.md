---
author: zjmantou
title: （4）Android NDK与JNI的区别有何不同？
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

**`以Android studio 4.0.2来分析讲解，gradle=6.1.1，如图文和网上其他资料不一致，可能是别的资料版本较低而已`**

---

## 1、NDK

Native Development Kit， Android的一个工具开发包。NDK集成了[交叉编译](https://so.csdn.net/so/search?q=%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91&spm=1001.2101.3001.7020 "https://so.csdn.net/so/search?q=%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91&spm=1001.2101.3001.7020")器(交叉编译器需要UNIX或LINUX系统环境)，并提供了相应的mk文件隔离CPU、平台、ABI等差异，开发人员只需要简单修改mk文件。

NDK能帮助开发者快速开发C、 C++的动态库，将动态库编译成.so文件供Java调用，并支持将.so和应用一起打包成 apk。

**为什么要使用NDK？**

1. 对于要求高性能的需求功能如游戏，采用C/C++运行的更加有效率
2. Java是半解释型语言，容易被反编译后得到源代码，而本地代码库.so不容易被反编译，安全程度高
3. 功能扩展性好，可以使用除了java的其他开发语言的开源库，不依赖于Dalvik Java虚拟机的设计
4. 便于平台移植，C/C++开发的代码可以移植到其他平台上使用

**`注意：`**

1. 一般应用用不上`NDK`，它会增大开发过程的复杂性。
2. `NDK`是属于 Android 的，与Java并无直接关系

---

## 2、.SO

NDK为了方便使用，提供了一些脚本，使得更容易的编译C/C++代码。在Android程序编译中会将C/C++编译成动态库.so，类似java库.jar文件一样，它的生成需要使用NDK工具来打包。如对.so文件如何生成感兴趣，请阅读我的另一文章 [《**Android Studio 4.0.+NDK .so库生成打包**》](https://blog.csdn.net/luo_boke/article/details/109362013 "https://blog.csdn.net/luo_boke/article/details/109362013")

so是shared object的缩写，见名思义就是共享的对象，机器可以直接运行的二进制代码。大到操作系统，小到一个专用软件，都离不开so，so主要存在于Unix和Linux系统中。实质so文件就是一堆C、C++的头文件和实现文件打包成一个库。

---

## 3、JNI

JNI的全称是Java Native Interface，即本地Java接口。因为 Java 具备跨平台的特点，所以Java 与 本地代码交互的能力非常弱。采用JNI特性可以增强 Java 与本地代码交互的能力，使Java和其他类型的语言如C++/C能够互相调用。

对于JNI的深入学习，请查阅另一篇博文[《Android JNI的深度进阶学习》](https://blog.csdn.net/luo_boke/article/details/109455910 "https://blog.csdn.net/luo_boke/article/details/109455910")

**`注意：`**

1. `JNI`是 Java 调用 Native 语言的一种特性，Java与C/C++交互
2. `JNI` 是属于 Java 的，与 Android 无直接关系，Android程序使用JNI比Java程序使用更简单

---

## 总结

1. NDK帮助将C/C++库打包成动态库.so，JNI是C/C++动态库能被调用的桥梁接口
2. JNI是java接口，用于Java与C/C++之间的交互，作为两者的桥梁
3. NDK是Android工具开发包，帮助快速开发C/C++动态库，相当于JDK开发java程序一样，同时能帮打包生成.so库

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