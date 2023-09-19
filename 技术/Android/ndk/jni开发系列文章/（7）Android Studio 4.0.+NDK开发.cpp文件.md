---
author: zjmantou
title: （7）Android Studio 4.0.+NDK开发.cpp文件
time: 2023-09-20 周三
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

**`以Android studio 4.0.2来分析讲解，所以是Android最新版NDK项目创建，其截图可能与低版本不一样。`**

---

## 1. 项目场景

我的Android Studio build:[gradle](https://so.csdn.net/so/search?q=gradle&spm=1001.2101.3001.7020 "https://so.csdn.net/so/search?q=gradle&spm=1001.2101.3001.7020")版本为4.0.2，gradle的版本为6.1.1，当我创建C++ project项目时，就会出现.cpp文件报红，如图：  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200012042.png)

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200013840.png)
  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200013127.png)


---

## 2. 问题描述：

虽然该报红不影响程序的运行，但是在进行JNI程序编写时不会自动提示，且无法再排序了。 对于有代码洁癖的我来说，报黄都不能忍了，岂能留着报红的地方。

以前Android Studio版本低时我创建过C++ Project程序没有该问题，那么确定肯定是配置哪里有问题了，只能各种测试了。

---

## 3. 分析过程：

多种方式尝试吧，撸起袖子就是一顿操作。

1. 删除.gradle、.idea、.cxx文件重新Sync
2. Invalidate Caches/Restart 重启Android Studio
3. 删除app.externalNativeBuild\cmake下的debug和release两个目录，别扯。Build 4.0.2 生成的Project根本就没这两文件。

**`以上方案都是通通不行的，最终找到是Cmake 3.10.2版本过高，与build版本不一致造成的`**

---

## 4. 解决方案：

1. 查看SDK Tools中CMake版本  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200013606.png)

2. 将CMake的版本降低  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200013134.png)

3. 问题解决
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200013374.png)


---

## 总结

对于Android 开发，降低配置版本是不可能的，这不是我的性格，对于报红报黄这类瑕疵，也是看不下去。所以即使可能不会影响程序的运行，我还是喜欢折腾下去直到找到解决办法。 这可能就是我的2B精神吧。

---

**相关链接**：

1. [JNI开发必学C++基础](https://blog.csdn.net/luo_boke/article/details/126908373 "https://blog.csdn.net/luo_boke/article/details/126908373")
2. [JNI开发必学C++使用实践](https://blog.csdn.net/luo_boke/article/details/126920916 "https://blog.csdn.net/luo_boke/article/details/126920916")
3. [Android Studio 4.0.+NDK项目开发详细教学](https://blog.csdn.net/luo_boke/article/details/107306531 "https://blog.csdn.net/luo_boke/article/details/107306531")
4. [Android NDK与JNI的区别有何不同？](https://blog.csdn.net/luo_boke/article/details/107234358 "https://blog.csdn.net/luo_boke/article/details/107234358")
5. [Android Studio 4.0.+NDK .so库生成打包](https://blog.csdn.net/luo_boke/article/details/109362013 "https://blog.csdn.net/luo_boke/article/details/109362013")
6. [Android JNI的深度进阶学习](https://blog.csdn.net/luo_boke/article/details/109455910 "https://blog.csdn.net/luo_boke/article/details/109455910")
7. [Android Studio 4.0.+NDK开发 This files is not part of the project](https://blog.csdn.net/luo_boke/article/details/109318721 "https://blog.csdn.net/luo_boke/article/details/109318721")

**博客书写不易，您的点赞、收藏是我前进的动力，请动动您的手指 ^ _ ^ !**