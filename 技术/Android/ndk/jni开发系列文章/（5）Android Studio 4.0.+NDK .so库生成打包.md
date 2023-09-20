---
author: zjmantou
title: （5）Android Studio 4.0.+NDK .so库生成打包
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

## 前言

涉及到一些算法或者底层驱动的时候，往往需要使用jni来开发。关于NDK和JNI如果还不了解，请查看我的另一篇博文《Android NDK与JNI的区别有何不同?》进行科普。

本篇博文主要是教学两种.so库的打包，稳文中有详细的图文指导，跟着一步步操作就能学会.so文件的打包，对于NDK开发学习请阅读我的《NDK开发系列》文章。

关于.so文件的生成有**两种方式**可以提供给大家参考，一种是CMake自动生成法，另一种是传统打包法。

---

## 1. 什么是.so库

NDK为了方便使用，提供了一些脚本，使得更容易的编译C/C++代码，这个编译文件为.so文件，它就C/C++库，类似java库.jar文件一样。so是shared object的缩写，见名思义就是共享的对象，机器可以直接运行的二进制代码。大到操作系统，小到一个专用软件，都离不开.so，.so主要存在于Unix和Linux系统中。

在Android开发中它的生成是需要使用JNI将C/C++文件打包成so库的，当然在其他开发软件中，由其他工具将其打包成so库。

.so文件在程序运行时就会加载，所以想使用Java调用.so文件，必有某个Java类运行时load了native库，并通过JNI调用了它的方法。

---

## 2. cmake生成.so方案

使用该种方案生成.so文件，需要先创建一个支持Cmake的 C++ Project，如果不会创建项目请阅读我的博文[《Android Studio 4.0.+NDK项目开发详细教学》](https://blog.csdn.net/luo_boke/article/details/107306531 "https://blog.csdn.net/luo_boke/article/details/107306531")

**1. 创建项目**  
根据Android Studio 引导创建一个native C++ Project

**2. 创建native函数**  
创建一个类，然后写一个native 函数，如：  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309192357863.png)

鼠标悬空报红的getData处，点击Create JNI function。。。  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309192358890.png)

在创建项目时，有自动生成一个native-lib.cpp文件，此时该文件中多了一个JNI getData函数  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309192358577.png)

完善JNI getData函数  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309192358518.png)


**3. 测试JNI函数**

```
public class MainActivity extends AppCompatActivity {
    static {
        System.loadLibrary("native-lib");
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Example of a call to a native method
        TextView tv = findViewById(R.id.sample_text);
        tv.setText(new MyNdkTest().getData());
    }
}
```

运行结果可见，JNI函数已被成功调用  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309192359406.png)


**4. 获取.so文件**  
将生成的.apk文件改为.zip文件，然后进行解压缩，就能看到.so文件，默认支持x86_64  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309192359845.png)


**5. 生成多类型.so文件**  
默认生成的是x86_64，如果想支持多种库架构，则可在app module的build.gradle中配置ndk支持。

```
android {
    .......
    defaultConfig {
    	...........
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
        externalNativeBuild {
            cmake {
                cppFlags ""
            }
        }
        ndk {
            // 设置支持的 SO 库构架，注意这里要根据你的实际情况来设置
            abiFilters 'armeabi-v7a', 'arm64-v8a', 'x86', 'x86_64'
        }
    }
    externalNativeBuild {
        cmake {
            path "src/main/cpp/CMakeLists.txt"
            version "3.6.0"
        }
    }
}
```

**6 .so文件测试**  
新建一个普通的Android程序，将库放入程序中运行  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309192359507.png)


1. 将生成的.so库放入lib文件夹中
2. 之前生成.so文件函数的类，在调用程序中依然需要相同的包名、文件名及方法名
3. 可以将库的加载放在java文件中，当程序启动时会自动加载.so类库  
    ![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309192359060.png)


**7. 小结**  
在Android Studio自动创建的native C++项目默认支持CMake方式，它支持JNI函数调用的入口在build.gradle中。
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200000282.png)

CMake的NDKx项目它有自己一套运行流程

1. Gradle 调用外部构建脚本CMakeLists.txt
2. CMake 按照构建脚本的命令将 C++ 源文件 native-lib.cpp 编译到共享的对象库中，并命名为 libnative-lib.so ，Gradle 随后会将其打包到APK中
3. 运行时，应用的MainActivity 会使用System.loadLibrary()加载原生库。应用就是可以使用库的原生函数getData()。

OK，自动生成.so库的方法就讲到这了，Android Studio帮我们自动化做了很多东西，所以so easy。 下面讲讲传统的.so库生成方案。

---

## 3. 传统生成.so方案

使用该种方案生成.so文件一定要先配置好NDK，如果不清楚如何配置NDK，请阅读一篇关于配置NDK的博文[《Android Studio 4.0.+NDK项目开发详细教学》](https://blog.csdn.net/luo_boke/article/details/107306531 "https://blog.csdn.net/luo_boke/article/details/107306531")。其打包.so文件的基本流程如下所述

**1. 在Java类中声明一个本地方法**

```
public class NDKTest {
    private native  int  count();
}
```
**Demo:**
```Java
package cn.com.javaproject.ndk;
public class JNIUtils {  
    // 加载native-jni  
    static {  
        System.loadLibrary("native-lib");  
    }    //java调C中的方法都需要用native声明且方法名必须和c的方法名一样  
    public native String stringFromJNI();  
}
```

---

**2. 执行指令javah获得C声明的.h文件**  
在terminal中cd 到\app\src\main\java目录下执行如下指令：  
`terminal可能出现不能用，则使用cmd命令行`

```
javah -encoding utf-8 -d ../jni -jni  com.xuanyuan.ndktest.NdKTest

//   javah:生成头文件指令
//   -encoding utf-8:编码格式 utf-8
//   -d ../jni:生成的文件放到与java目录同级的jni文件中，jni文件若不存在会自动创建
//  -jni：当前目录下生成.h文件,当前目录是cd进入的目录，这里是\app\src\main\java
//  com.xuanyuan.ndktest.NdKTest:包名+类名
```

指命令后会在…\app\src\main\jni目录下生成一个com_xuanyuan_ndktest_NdKTest.h文件

```
/* DO NOT EDIT THIS FILE - it is machine generated */
#include <jni.h>
/* Header for class com_xuanyuan_ndktest_NdKTest */

#ifndef _Included_com_xuanyuan_ndktest_NdKTest
#define _Included_com_xuanyuan_ndktest_NdKTest
#ifdef __cplusplus
extern "C" {
#endif
/*
 * Class:     com_xuanyuan_ndktest_NdKTest
 * Method:    test
 * Signature: ()Ljava/lang/String;
 */
JNIEXPORT jstring JNICALL Java_com_xuanyuan_ndktest_NdKTest_test
  (JNIEnv *, jobject);

#ifdef __cplusplus
}
#endif
#endif
```

**Demo:cn_com_javaproject_ndk_JNIUtils.h**
```c
/* DO NOT EDIT THIS FILE - it is machine generated */  
#include <jni.h>  
/* Header for class cn_com_javaproject_ndk_JNIUtils */  
  
#ifndef _Included_cn_com_javaproject_ndk_JNIUtils  
#define _Included_cn_com_javaproject_ndk_JNIUtils  
#ifdef __cplusplus  
extern "C" {  
#endif  
/*  
 * Class:     cn_com_javaproject_ndk_JNIUtils * Method:    stringFromJNI * Signature: ()Ljava/lang/String; */JNIEXPORT jstring JNICALL Java_cn_com_javaproject_ndk_JNIUtils_stringFromJNI  
  (JNIEnv *, jobject);  
  
#ifdef __cplusplus  
}  
#endif  
#endif
```


另一种方式是使用External tools工具，使用其javah指令，该指令需要我们自行配置，后面会单独讲解。  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200000758.png)


---

**3. 获得.c文件并实现本地方法**  
生成的.h文件中函数相当于是一个抽象方法，具体实现需要我们来自定义。此时在jni中重建一个demo.c文件，将com_xuanyuan_ndktest_NdKTest.h中的完全复制过来，将函数完整实现。  
`自己实现的C++方法要写对，不太熟悉C++的人找C++工程师支援，否则无法制作.so文件`

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200000348.png)

**Demo：**
```c
#include "cn_com_javaproject_ndk_JNIUtils.h"  
/**  
 * 上边的引用标签一定是.h的文件名家后缀，方法名一定要和.h文件中的方法名称一样  
 */JNIEXPORT jstring JNICALL Java_cn_com_javaproject_ndk_JNIUtils_stringFromJNI  
        (JNIEnv *env, jobject ojb){  
    return (*env) -> NewStringUTF(env,"Hello, I'm from jni");  
}
```

**4. 创建Android.mk和Application.mk**  
在jni目录中创建Android.mk和Application.mk两文件，并配置其参数,两个文件如不编写或编写正常会出现报错。

```
//Android.mk 参数

//设置工作目录，它用于在开发tree中查找源文件。宏my-dir由Build System提供,会返回Android.mk文件所在的目录
LOCAL_PATH := $(call my-dir)

//CLEAR_VARS变量由Build System提供。指向一个指定的GNU Makefile，由它负责清理LOCAL_xxx类型文件，但不是清理LOCAL_PATH
//所有的编译控制文件由同一个GNU Make解析和执行，其变量是全局的。所以清理后才能便面相互影响,这一操作必须有
include $(CLEAR_VARS)

// LOCAL_MODULE模块必须定义，以表示Android.mk中的每一个模块，名字必须唯一且不包含空格
//Build System 会自动添加适当的前缀和后缀。例如，demo，要生成动态库，则生成libdemo.so。但请注意：如果模块名字被定义为libabd，则生成libabc.so。不再添加前缀。
LOCAL_MODULE := DEMO


// 指定参与模块编译的C/C++源文件名。不必列出头文件，build System 会自动帮我们找出依赖文件。缺省的C++ 源码的扩展名为.cpp。
LOCAL_SRC_FILES := demo.c


// BUILD_SHARED_LIBRARY是Build System提供的一个变量，指向一个GUN Makefile Script。它负责收集自从上次调用include $(CLEAR_VARS)后的所有LOCAL_xxxxinx。并决定编译什么类型
//1. BUILD_STATIC_LIBRARY：编译为静态库
//2. BUILD_SHARED_LIBRARY：编译为动态库
//3. BUILD_EXECUTABLE：编译为Native C 可执行程序
//4. BUILD_PREBUILT：该模块已经预先编译
include $(BUILD_SHARED_LIBRARY)



**************************************************
#Application.mk 参数
// 默认生成支持的多种类型.so
APP_ABI := all

//APP_PLATFORM := android-16不配置,打包.so会出错
APP_PLATFORM := android-16
```

**Demo:Android.mk**
```C
LOCAL_PATH := $(call my-dir)  
include $(CLEAR_VARS)  
LOCAL_MODULE := native-lib  
LOCAL_SRC_FILES := native-lib.c  
include $(BUILD_SHARED_LIBRARY)
```

**Demo:Application.mk**
```C
APP_ABI := all    
APP_PLATFORM := android-24
```

如果我们需要进行多个C++文件一起编译或者已有的.so参与编译，这里给出配置示例

```
# Android.mk
    // 示例1：单个c++文件参与编译
    include $(CLEAR_VARS)
    // 编译出来的文件名
    LOCAL_MODULE := DEMO
    LOCAL_SRC_FILES := demo.c

    // 多个c++文件参与编译
    include $(CLEAR_VARS)
    // 编译出来的文件名
    LOCAL_MODULE := DEMO
    LOCAL_SRC_FILES := \
    		test1.cpp \
      		test/ftest2.cpp \
       		test3.c \

// 示例2：.so文件参与模块编译
    include $(CLEAR_VARS)
     // 编译出来的文件名
    LOCAL_MODULE := DEMO2
    LOCAL_SRC_FILES := test/DEMO.so
    // 设置可被依赖
    include $(PREBUILT_SHARED_LIBRARY)
```

---

**5. 打包.so库**  
各种文件准备好后，cd到\app目录下，执行命令 ndk-build即可，我没有用terminal,不知啥原因用不了。  
执行命令后会在libs文件夹中生成.so文件，默认支持4种命令库。

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200001798.png)

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200001355.png)


同样另一种方式是使用External tools工具，使用其ndk-build指令，该指令需要我们自行配置，后面会单独讲解。  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200002275.png)


---

**6. 测试.so库**  
测试.so库生成完毕，正常可用。  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200002566.png)


---

## 4. external tools配置

在上面制作.h文件和.so文件中要在cmd或者terminal中输入javah、ndk-build命令比较麻烦，我们可以在external tools中进行配置，一次配置，终生幸福！  

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200002277.png)


**1. javah配置**  

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200002180.png)


```
//javah.exe的地址
Program：$JDKPath$\bin\javah
//生成.h文件的路径指定在jni文件中，$FileClass$为源.java文件
Arguments：-encoding UTF-8 -d ../jni -jni $FileClass$
//进行编译成.h文件的源.java文件目录
Working directory：$ProjectFileDir$\app\src\main\java

```

**2. ndk-build配置**  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200003917.png)


```
//ndk-build.cmd路径
Program：$ModuleSdkPath$/ndk/21.3.6528147/ndk-build.cmd
//生成的.so文件存放位置
Arguments：NDK_LIBS_OUT=$ProjectFileDir$\app\libs
//编译源文件的目录
Working directory：$ProjectFileDir$\app\src\main

```

---

## 总结

本篇博文主要讲解了两种.so库打包生成的方式，对于NDK开发的其他知识点学习请继续阅读我的系列文章。

`在我们使用.so文件时，一定要记得做好配置，否则会出现无法找到.so库的异常`

```
android {
    compileSdkVersion 30
    buildToolsVersion "30.0.2"
    ...
    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }
}
```

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