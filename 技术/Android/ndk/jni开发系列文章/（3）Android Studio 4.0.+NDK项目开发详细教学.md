---
author: zjmantou
title: （3）Android Studio 4.0.+NDK项目开发详细教学
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


**`以Android studio 4.0.2来分析讲解，所以是Android最新版NDK项目创建，其截图可能与低版本不一样。`**

---

## 前言

涉及到一些算法或者底层驱动的时候，往往需要使用jni来开发。关于NDK和JNI如果还不了解，请查看我的另一篇博文[《Android NDK与JNI的区别有何不同？》](https://blog.csdn.net/luo_boke/article/details/107234358 "https://blog.csdn.net/luo_boke/article/details/107234358")进行科普，创建NDK项目开干。

---

## NDK环境配置

想要使用Android Studio 进行NDK开发，先要进行配置。

1. 安装NDK  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309192348188.png)

2. 配置NDK环境  
   ![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309192348639.png)
  
    复制NDK的路径进入系统环境变量中  
    ![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309192348426.png)

3. 环境配置成功  
    然后打开cmd，输入ndk-build，可见环境已经配置好了。  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309192349665.png)


---

## NDK项目创建

使用Android Sudio创建DNK项目的方式有两种：一种是直接创建C++ project；另一种是在Java 项目中手动配置，虽然更麻烦些，但是更灵活值得学习。

现在官方推荐使用CMake工具来开发jni，cmake方式开发jni项目相对更简单易上手。可以将NDK类别为SDK,将Cmake类别为build，将它看作一个编译类工具。

---

新建项目直接勾选C++ project进行项目创建。  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309192349731.png)

创建完项目后自动生成的.cpp文件会报红，一般方法解决不了该问题，请阅读我的另一博文[《Android Studio 4.0+NDK开发 This files is not part of the project》](https://blog.csdn.net/luo_boke/article/details/109318721 "https://blog.csdn.net/luo_boke/article/details/109318721")  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309192350350.png)


---

## 工程文件说明

用Android Studio 创建的默认Native C++项目默认有三个比较特殊的文件CMakeLists.txt、native-lib.cpp、MainActivty。这三个文件下面我们一一详细说明  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309192350509.png)

除了以上三个文件，还有app build.gradle文件配置有差异外，其他地方与我们创建的普通Android 项目并无差异。  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309192351030.png)

**注意：在Build 4.+以后，CMakeLists.txt的路径在src/main/cpp路径下**

---

**1. CMakeLists.txt文件**

```
# Sets the minimum version of CMake required to build the native library.
# 设置构建本机库所需的CMake最低版本。
cmake_minimum_required(VERSION 3.4.1)

# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds them for you.
# Gradle automatically packages shared libraries with your APK.
# 创建并命名一个库，将其设置为STATIC或SHARED，并提供其源代码的相对路径。 
# 您可以定义多个库，CMake会为您构建它们。Gradle自动将共享库与您的APK打包在一起。
add_library( # Sets the name of the library.
             #库的名称
             native-lib

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             # 库所在位置的相对路径
             native-lib.cpp )

# Searches for a specified prebuilt library and stores the path as a
# variable. Because CMake includes system libraries in the search path by
# default, you only need to specify the name of the public NDK library
# you want to add. CMake verifies that the library exists before
# completing its build.
# 搜索指定的预构建库并将路径存储为变量。由于默认情况下CMake在搜索路径中包括系统库，因此您只需要指定要添加的公共NDK库的名称即可。
# 在完成构建之前，CMake会验证该库是否存在。
find_library( # Sets the name of the path variable.
              log-lib

              # Specifies the name of the NDK library that
              # you want CMake to locate.
              log )

# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.

target_link_libraries( # Specifies the target library.
                       native-lib

                       # Links the target library to the log library
                       # included in the NDK.
                       ${log-lib} )
```

**2. native-lib.cpp**  
在创建C++ project时默认生成的示例调用文件，包含了jni代码

**3. MainActivty**

```
// Used to load the 'native-lib' library on application startup.
    static {
        System.loadLibrary("native-lib");
    }
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Example of a call to a native method
        TextView tv = findViewById(R.id.sample_text);
        tv.setText(stringFromJNI());
    }
    public native String stringFromJNI();
```

1. 静态代码块中的代码在应用程序启动时就会加载native-lib 本地库
2. Java代码通过stringFromJNI()本地等方法接口调用C++函数

补充：如果是对旧有项目进行支持NDK，只需要将上面三个文件对应添加，将build.gradle配置一下即可。

---

## NDK调用流程

运行程序如下图，发现在MainActivity中已能调用Jni代码中的C++代码了。我们整理下JNI的运行流程。  
![在这里插入图片描述](file:///Users/mantou/.config/joplin-desktop/resources/c1e529ba50e7490b85afe654dc7a995b.png)  
**运行流程：**

1. 在编译运行时，Gradle调用了CMakeLists.txt文件，将C++源文件native-lib.cpp编译到共享对象中，并命名为libnative-lib.so
2. Gradle将libnative-lib.so与apk打包在一起。将生成的apk解压是可以得到证明的  
    ![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309192351638.png)

3. 在程序运行时，由于System.loadLibrary()是静态代码块中调用的，所以在程序加载后libnative-so 已被加载
4. 在Java程序中调用stringFromJNI()方法，通过JNI接口调用了C++中的stringFromJNI()函数。

---

## 总结

1. Android运行在Dalvik或者ART虚拟机上，执行.dex文件的Java代码，其实虚拟机上也能允许C/C++代码。
    
2. 在虚拟机中有一层Native层，里面有很多Android的核心代码，且这些代码使用C/C++写的。我们看源码时追踪代码最终源头发现很多都是native申明的方法。在这其实已经使用到了JNI/NDK技术了。
    
3. NDK是Android 的开发工具包，如同SDK是帮助Android开发者开发Java程序一样，NDK能帮助Android开发者快速开发C/C++程序。它在开发中提供对C/C++的的自动提示和编译检测。
    
4. 通过NDK工具可以将C/C++动态库编译成.so文件，可以将.so文件与应用程序一起打包成apk
    
5. NDK是采用JNI机制实现的,它可以将C/C++代码编译到原生库中
    

**本篇博文主要讲解了使用Android Studio 4.0.+创建NDK项目的详细过程，对于碰到的问题进行解决和项目结构的讲解。**

**对于NDK开发的进阶学习请继续阅读我的下一篇博文。**

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