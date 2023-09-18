---
author: zjmantou
title: Android.mk上手指南
time: 2023-09-18 周一
tags:
  - Android
  - makefile
---
Android.mk是Android源码中提供的一套用于编译Android系统、子模块的基于makefile语法规则的脚本文件。作为一名Android系统工程师，我们必须要了解Android.mk的语法规则，这样才能得心应手的修改Android系统。

一个工程中的源文件不计其数，其按类型、功能、模块分别放在若干个目录中，makefile定义了一系列的规则来指定哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作，因为 makefile就像一个[Shell脚本](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2FShell%25E8%2584%259A%25E6%259C%25AC%2F572265)一样，也可以执行操作系统的[命令](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E5%2591%25BD%25E4%25BB%25A4%2F13020279) 。《百度百科：makefile》

虽然Android 7.0 之后Google提供了一种基于新语法结构Android.bp脚本来替代语法结构繁杂的Android.mk，但是即使是Android 10 中依然有大量模块还在使用Android.mk进行构建，预计今后一段时间内，Android.mk与Android.bp依然会同时存在于Android源码中。有关Android.bp的使用，请参考这里[Android.bp入门教程](https://www.jianshu.com/p/f23e18933122)。

## 一、最基础的Android.mk构成

在详细了解语法之前，最好先了解 Android.mk 文件所含内容的基本信息。为此，我们先从一个最简单的Android.mk入手，来逐步了解mk的语法结构。

```
LOCAL_PATH := $(call my-dir)   
include $(CLEAR_VARS)   
LOCAL_SRC_FILES := hello-jni.c 
LOCAL_MODULE    := hello-jni     
include $(BUILD_SHARED_LIBRARY)  
```

Android.mk 文件必须先定义 LOCAL_PATH 变量

```
LOCAL_PATH := $(call my-dir)
```

CLEAR_VARS 变量指向一个特殊的 GNU Makefile，后者会为您清除许多 LOCAL_XXX 变量，例如 `LOCAL_MODULE`、`LOCAL_SRC_FILES` 和 `LOCAL_STATIC_LIBRARIES`。请注意，GNU Makefile 不会清除 `LOCAL_PATH`。此变量必须保留其值，因为系统在单一 GNU Make 执行上下文（其中的所有变量都是全局变量）中解析所有构建控制文件。在描述每个模块之前，您必须声明（重新声明）此变量。

声明`LOCAL_SRC_FILES`变量

```
LOCAL_SRC_FILES := hello-jni.c 
```

构建系统生成模块时所用的源文件，可以是单个源文件，也可以是一个文件夹。

声明`LOCAL_MODULE`变量

```
LOCAL_MODULE := hello-jni 
```

`LOCAL_MODULE` 变量存储您要构建的模块的名称。请在应用的每个模块中使用一次此变量。每个模块名称必须唯一，且不含任何空格。构建系统在生成最终共享库文件时，会对您分配给 `LOCAL_MODULE` 的名称自动添加正确的前缀和后缀。例如，上述示例会生成名为 libhello-jni.so 的库。

**注意**：如果模块名称的开头已经是 lib，构建系统不会添加额外的 lib 前缀；而是按原样采用模块名称，并添加 .so 扩展名。因此，比如原来名为 libfoo.c 的源文件仍会生成名为 libfoo.so 的共享对象文件。此行为是为了支持 Android 平台源文件根据 Android.mk 文件生成的库；所有这些库的名称都以 lib 开头。

LOCAL_SRC_FILES 变量必须包含要构建到模块中的源文件列表。

声明`include $(BUILD_SHARED_LIBRARY)`

```
include $(BUILD_SHARED_LIBRARY)   
```

通过`include $(BUILD_SHARED_LIBRARY)` 编译工具，会将上面设定的一切连接到一起`BUILD_SHARED_LIBRARY` 变量指向一个 GNU Makefile 脚本，该脚本会收集您自最近 include 以来在 `LOCAL_XXX` 变量中定义的所有信息。此脚本确定要构建的内容以及构建方式。

上面这些就组成了一个最基础的Android.mk，下面我们继续介绍Android.mk中出现的各种关键字含义和用法。

## 二、模块

在Android.mk中我们把以`include $(CLEAR_VARS)`标记开头，到`include $(BUILD_XXX)`标记结束，这中间描述的所有行为，称为一个模块。

例如：下列mk中包含了，两个模块。

- 模块一：源码第2行至第9行，用于编译一个java类库
- 模块二：源码第12行至第26行，用于编译一个apk安装包

```
LOCAL_PATH:= $(call my-dir) 
include $(CLEAR_VARS) 
LOCAL_SRC_FILES := \         
		$(call all-logtags-files-under, src) 
LOCAL_MODULE := settings-logtags 
## 构建一个 java 类库 
include $(BUILD_STATIC_JAVA_LIBRARY) 

# 构建一个apk文件 
include $(CLEAR_VARS) LOCAL_PACKAGE_NAME := Settings 
#省略其它描述... 
LOCAL_USE_AAPT2 := true 

LOCAL_SRC_FILES := $(call all-java-files-under, src) 

LOCAL_STATIC_ANDROID_LIBRARIES := \     
		androidx-constraintlayout_constraintlayout \ 
#省略其它描述... 
include frameworks/base/packages/SettingsLib/common.mk 
include frameworks/base/packages/SettingsLib/search/common.mk 

include $(BUILD_PACKAGE) 
```

## 三、include变量

### CLEAR_VARS

用于取消位于`include $(CLEAR_VARS)`之前定义的所有的LOCAL_XXX变量中定义的值，但是LOCAL_PATH中定义的值不会被取消。  
在描述新的模块前，需要用include包含此变量。

### BUILD_JAVA_LIBRARY

`include $(BUILD_JAVA_LIBRARY)`用于构建java类库，该类库中的代码会以dex的形式存在，如图所示  
  

![](https://cdn.nlark.com/yuque/0/2022/png/26044650/1670147783642-b1164d2c-9cbd-4d67-ab99-d7b80346e22d.png)

### BUILD_STATIC_JAVA_LIBRARY

`include $(BUILD_STATIC_JAVA_LIBRARY)`用于构建java类库，该类库中代码会以class文件的形式存在.  
如果编译出jar是用于app的开发，应该使用该变量描述。

### BUILD_PACKAGE

`include $(BUILD_PACKAGE)`用于构建Android 应用程序安装包。

### BUILD_MULTI_PREBUILT

`include $(BUILD_MULTI_PREBUILT)`用于构建预制库，这些库一般可以指定在lib文件夹下。

### BUILD_STATIC_LIBRARIES

`include $(BUILD_STATIC_LIBRARIES)`用于构建native静态库。

## 四、模块描述变量

### LOCAL_PATH

此变量用于指定当前文件的路径。必须在 `Android.mk` 文件开头定义此变量。以下示例演示了如何定义此变量：

```
LOCAL_PATH := $(call my-dir) 
```

`CLEAR_VARS` 所指向的脚本不会清除此变量。因此，即使 `Android.mk` 文件描述了多个模块，也只需定义此变量一次。

### LOCAL_MODULE

此变量用于设定模块的名称。指定的名称在所有模块名称中必须唯一，并且不得包含任何空格。必须先定义该名称，然后才能添加其它的脚本（`CLEAR_VARS` 的脚本除外）。如下定义在配合`include $(BUILD_STATIC_JAVA_LIBRARY)`使用时会编译时生成一个`libsettings-logtags`的类库，lib是编译时系统自动加上的。

```
LOCAL_MODULE := settings-logtags 
```

### LOCAL_MODULE_FILENAME

此变量能够替换构建系统为其生成的文件默认使用的名称。例如，如果 `LOCAL_MODULE` 的名称为 `settings-logtags`，你可以强制系统将其生成的文件命名为 `settingslib`。

```
LOCAL_MODULE := settings-logtags LOCAL_MODULE_FILENAME := settingslib 
```

### LOCAL_SRC_FILES

此变量包含构建系统生成模块时所用的源文件列表。

```
LOCAL_SRC_FILES := \         
		$(call all-logtags-files-under, src) 
```

### LOCAL_PACKAGE_NAME

此变量用于指定编译后生成的Android APK的名字。例如，如下方式，会生成一个Settings.apk的文件。

```
LOCAL_PACKAGE_NAME := Settings 
```

### LOCAL_CERTIFICATE

此变量用于指定APK的签名方式。如果不指定，默认使用testkey签名。

```
LOCAL_CERTIFICATE := platform 
```

Android中共有四中签名方式：

- testkey：普通APK，默认使用该签名。
- platform：该APK完成一些系统的核心功能。经过对系统中存在的文件夹的访问测试，这种方式编译出来的APK所在进程的UID为system。
- shared：该APK需要和home/contacts进程共享数据。
- media：该APK是media/download系统中的一环。

### LOCAL_PRODUCT_MODULE

为true表示将此apk安装到priv-app目录下。

```
LOCAL_PRODUCT_MODULE := true 
```

### LOCAL_SDK_VERSION

标记SDK 的version 状态。取值范围有四个`current` `system_current` `test_current` `core_current` 。

```
LOCAL_SDK_VERSION := current 
```

### LOCAL_PRIVATE_PLATFORM_APIS

设置后，会使用sdk的hide的api?肀嘁搿１嘁氲牧PK中使用了系统级API，必须设定该值。

```
LOCAL_PRIVATE_PLATFORM_APIS := true 
```

### LOCAL_USE_AAPT2

此值用于设定是否开启AAPT2打包APK，AAPT是Android Asset Packaging Tool的缩写，AAPT2在AAPT的基础做了优化。

```
LOCAL_USE_AAPT2 := true 
```

### LOCAL_STATIC_ANDROID_LIBRARIES

此值用于设定依赖的静态Android库

```
LOCAL_STATIC_ANDROID_LIBRARIES := \     
	androidx-constraintlayout_constraintlayout \     
	androidx.slice_slice-builders \ 
```

### LOCAL_JAVA_LIBRARIES

此值用于设定依赖的共享java类库。`LOCAL_JAVA_LIBRARIES`引用的外部Java库在编译时可以找到相关的东西，但并不打包到本模块，在runtime时需要从别的地方查找，这个别的地方就是在编译时将引用的外部Java库的模块名添加到`PRODUCT_BOOT_JARS`。

```
LOCAL_JAVA_LIBRARIES := \     
	telephony-common \     
	ims-common 
```

### LOCAL_STATIC_JAVA_LIBRARIES

此值用于设定依赖的静态java类库。`LOCAL_STATIC_JAVA_LIBRARIES`会把引用的外部Java库直接编译打包到本模块中，在runtime时可以直接从本模块中找到相关的jar。

```
LOCAL_STATIC_JAVA_LIBRARIES := \     
	androidx-constraintlayout_constraintlayout-solver \     
	androidx.lifecycle_lifecycle-runtime \     
	androidx.lifecycle_lifecycle-extensions \ 
```

### LOCAL_PROGUARD_FLAG_FILES

此值用于设定混淆的标志。

```
LOCAL_PROGUARD_FLAG_FILES := proguard.flags 
```

Android.mk中以LOCAL_XXX开头的描述变量非常多，这里只列举了一些常用的变量。

## 五、函数宏

Android系统中提供了大量的宏函数，使用 `$(call <function>)` 可以对其进行求值，返回文本信息。

### my-dir

这个宏返回最后包括的 makefile 的路径，通常是当前 `Android.mk` 的目录。`my-dir` 可用于在 `Android.mk` 文件开头定义 `LOCAL_PATH`。例如：

```
LOCAL_PATH := $(call my-dir) 
```

由于 GNU Make 的工作方式，这个宏实际返回的是构建系统解析构建脚本时包含的最后一个 makefile 的路径。因此，包括其他文件后就不应调用 `my-dir`。

例如：

```
LOCAL_PATH := $(call my-dir) 
# ... declare one module 
include $(LOCAL_PATH)/foo/`Android.mk` 
LOCAL_PATH := $(call my-dir) 
# ... declare another module 
```

这里的问题在于，对 `my-dir` 的第二次调用将 `LOCAL_PATH` 定义为 `$PATH/foo`，而不是 `$PATH`，因为这是其最近的 include 所指向的位置。

在 `Android.mk` 文件中的任何其他内容后指定额外的 include 可避免此问题。例如：

```
LOCAL_PATH := $(call my-dir) 
# ... declare one module 
LOCAL_PATH := $(call my-dir) 
# ... declare another module 
# extra includes at the end of the Android.mk file 
include $(LOCAL_PATH)/foo/Android.mk 
```

如果以这种方式构造文件不可行，请将第一个 `my-dir` 调用的值保存到另一个变量中。例如：

```
MY_LOCAL_PATH := $(call my-dir) 
LOCAL_PATH := $(MY_LOCAL_PATH) 
# ... declare one module 
include $(LOCAL_PATH)/foo/`Android.mk` 
LOCAL_PATH := $(MY_LOCAL_PATH)
#  ... declare another module 
```

### all-java-files-under

返回位于`<name>`目录下的所有java文件。如果不指定`<name>`，怎么返回`my-dir`目录下所有的java文件。

```
include $(call all-java-files-under,<name>) 
```

### all-makefiles-under

返回位于当前 `<name>` 路径下所有目录中的 Android.mk 文件列表。利用此函数，可以为构建系统提供深度嵌套的源目录层次结构。默认情况下，系统只在 Android.mk 文件所在的目录中查找文件。

```
include $(call all-makefiles-under,<name>) 
```

## 六、常见疑问

### 引入lib文件夹下的第三方库

在实际开发中，我们经常需要在app中引入第三方的jar或aar，在Android.mk中，可以按照如下的方式描述：

```
LOCAL_PATH:= $(call my-dir) 
include $(CLEAR_VARS) 
# ... LOCAL_STATIC_JAVA_LIBRARIES := \     
	contextualcards # 给引入的jar或aar 起一个别名 
# ... 
include $(BUILD_PACKAGE) 
# ====  预制类库 ======================== 
include $(CLEAR_VARS) LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES := \     
	contextualcards:libs/contextualcards.aar # 指定contextualcards的实际路径 
include $(BUILD_MULTI_PREBUILT) 
```

这里需要特别注意一点：_在android.mk中引入aar，编译时只会引入aar包中的java文件，资源文件如：icon、xml等都不会被编译到apk中_。

### 引入AndroidX库

开发过程如果想要引入AndroidX的类库可以参考下面的方式编写。

```
LOCAL_PATH:= $(call my-dir) 
include $(CLEAR_VARS) 
# ... 
LOCAL_STATIC_ANDROID_LIBRARIES := \     
	androidx-constraintlayout_constraintlayout \     
	androidx.appcompat_appcompat \ 

LOCAL_STATIC_JAVA_LIBRARIES := \     
	androidx-constraintlayout_constraintlayout-solver \ 
# ... 
include $(BUILD_PACKAGE) 
# ... 
```

上面的mk中主要是引入了`AndroidX`下的`appcompat`和`constraintlayout`库。其中`constraintlayout`不仅需要在`LOCAL_STATIC_ANDROID_LIBRARIES`引入`androidx-constraintlayout_constraintlayout`，还需要在`LOCAL_STATIC_JAVA_LIBRARIES`中引入`androidx-constraintlayout_constraintlayout-solver`

在控制台使用指令 find prebuilts/sdk/ -name Android.bp|xargs grep "name._粗略的名字"，查询类库的引入方式。使用之前需要先 # source build/envsetup.sh # lunch，不然无法执行find指令。  
__使用示例：find prebuilts/sdk/ -name Android.bp|xargs grep "name._constraintlayout"

## 七、参考『系统设置』的Android.mk

Android.mk的编写，我们很多时候可以参考系统中已有的Android.mk，例如下面给的就是Android R中系统设置的Android.mk。

该mk的位于 packages/apps/Settings 下

```
LOCAL_PATH:= $(call my-dir) 
include $(CLEAR_VARS) LOCAL_SRC_FILES := \         
	$(call all-logtags-files-under, src) 

LOCAL_MODULE := settings-logtags 
# 构建一个 libsettings-logtags的静态java类库 
include $(BUILD_STATIC_JAVA_LIBRARY) 

# 构建一个Settings.apk 
include $(CLEAR_VARS) 

LOCAL_PACKAGE_NAME := Settings 
LOCAL_PRIVATE_PLATFORM_APIS := true 
LOCAL_CERTIFICATE := platform 
LOCAL_PRODUCT_MODULE := true 
LOCAL_PRIVILEGED_MODULE := true 
LOCAL_REQUIRED_MODULES := privapp_whitelist_com.android.settings 
LOCAL_MODULE_TAGS := optional 
LOCAL_USE_AAPT2 := true 

LOCAL_SRC_FILES := $(call all-java-files-under, src) 

LOCAL_STATIC_ANDROID_LIBRARIES := \     
	androidx-constraintlayout_constraintlayout \     
	androidx.slice_slice-builders \     
	androidx.slice_slice-core \     
	androidx.slice_slice-view \     
	androidx.core_core \     
	androidx.appcompat_appcompat \     
	androidx.cardview_cardview \     
	androidx.preference_preference \     
	androidx.recyclerview_recyclerview \     
	com.google.android.material_material \     
	setupcompat \     
	setupdesign 

LOCAL_JAVA_LIBRARIES := \     
	telephony-common \     
	ims-common 

LOCAL_STATIC_JAVA_LIBRARIES := \     
	androidx-constraintlayout_constraintlayout-solver \     
	androidx.lifecycle_lifecycle-runtime \     
	androidx.lifecycle_lifecycle-extensions \     
	guava \     
	jsr305 \     
	settings-contextual-card-protos-lite \     
	settings-log-bridge-protos-lite \     
	contextualcards \     
	settings-logtags \     
	zxing-core-1.7 

LOCAL_PROGUARD_FLAG_FILES := proguard.flags

ifneq ($(INCREMENTAL_BUILDS),)    
	LOCAL_PROGUARD_ENABLED := disabled     
	LOCAL_JACK_ENABLED := incremental     
	LOCAL_JACK_FLAGS := --multi-dex native 
endif 

include frameworks/base/packages/SettingsLib/common.mk 
include frameworks/base/packages/SettingsLib/search/common.mk 

include $(BUILD_PACKAGE) 

# ====  prebuilt library  ======================== 
include $(CLEAR_VARS) 

LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES := \     
	contextualcards:libs/contextualcards.aar 
include $(BUILD_MULTI_PREBUILT) 

# Use the following include to make our test apk. 
ifeq (,$(ONE_SHOT_MAKEFILE)) 
include $(call all-makefiles-under,$(LOCAL_PATH)) 
endif 
```

## 八、参考资料

[Android.mk | Android NDK | Android Developers (google.cn)](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Fndk%2Fguides%2Fandroid_mk)  
[编译_tanliyin的博客-CSDN博客](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Ftanliyin%2Farticle%2Fdetails%2F72883647)  
[Android.mk 使用 环境 小结_快乐&&平凡-CSDN博客](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fwh_19910525%2Farticle%2Fdetails%2F8265824)

  
作者：林栩  
链接：https://www.jianshu.com/p/99442ac2de0c  
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。