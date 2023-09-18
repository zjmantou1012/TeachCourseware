---
author: zjmantou
title: （2）集成第三方APK
time: 2023-09-18 周一
tags:
  - Android
  - 车联网
---


## 前言

在车载的应用开发过程中，会有一类特殊的需求，就是在预装一些第三方app，常见的有百度地图车载版、车载微信等等。这类app OEM 厂商都不会得到源码，只能得到一个apk。

本篇文章基于Android R演示如何在aosp_car_x86_x64中预装第三方apk。aosp_car_x86_x64我们在编译AOSP选择的build_type，如果你还不知道如何编译AOSP可以参考这篇文章[[（1）- Android Automotive概述与编译]]。

各个OEM厂商预装第三方APK的方式其实都有不同，本文只演示相对简单的一种方式。

## 1. 应用安装的目录

**/system/priv-app**  
该路径存放一些系统底层的应用，比如Setting，systemUI等。该目录中的app拥有较高的系统权限，而且如果要使用`android:protectionLevel=signatureOrSystem`，那么该app必须放到priv-app目录中去。

**/system/app**  
该目录中存放的系统app权限相对较低，而且当拥有root权限时，就有可能卸载掉这些app。

**/vendor/app**  
该目录存放vendor厂商的app

**/oem/app**  
该目录中存放oem特有的app。

**/data/app**  
用户安装的第三方app。

**PMS启动的时候，也是按照上述顺序逐个扫描解析这些目录中的apk的**

## 2. 预装APK

在 `device/generic/car` 新建文件夹，如`bilibili`将APK拷贝到bilibili文件夹下，创建Android.mk文件，内容如下：

```bash
LOCAL_PATH:=$(call my-dir)
include $(CLEAR_VARS)
LOCAL_SRC_FILES := bilibili.apk
LOCAL_MODULE_CLASS := APPS
#可以为user、eng、tests、optional，optional代表在任何版本下都编译
LOCAL_MODULE_TAGS := optional
#编译模块的名称
LOCAL_MODULE := bilibili
#可以为testkey、platform、shared、media、PRESIGNED（使用原签名），platform代表为系统应用
LOCAL_CERTIFICATE := PRESIGNED
#应用输出路径，此处为system/app
LOCAL_MODULE_PATH := $(TARGET_OUT)/app
#不设置或者设置为false，安装位置为system/app，如果设置为true，则安装位置为system/priv-app?
LOCAL_PRIVILEGED_MODULE := false
#module的后缀，可不设置
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)
# 关闭预编译，不会生成OAT文件
LOCAL_DEX_PREOPT := true
include $(BUILD_PREBUILT)
```

将bilibili添加到device/generic/car/aosp_car_x86_64.mk

```bash
PRODUCT_PACKAGE_OVERLAYS := device/generic/car/common/overlay

EMULATOR_VENDOR_NO_SENSORS := true
$(call inherit-product, device/generic/car/emulator/aosp_car_emulator.mk)
$(call inherit-product, $(SRC_TARGET_DIR)/product/aosp_x86_64.mk)

# 将 packages 添加到 PRODUCT_PACKAGES 中
PRODUCT_PACKAGES += \
  bilibili \

EMULATOR_VENDOR_NO_SOUND := true
PRODUCT_NAME := aosp_car_x86_64
PRODUCT_DEVICE := generic_car_x86_64
PRODUCT_BRAND := Android
PRODUCT_MODEL := Car on x86_64 emulator
```

重新编译整个工程

```bash
# 清除out目录下对应板文件夹中的内容
make installclean
make -j8
```

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670158030321-9aa09f98-2197-4f8e-86d5-ec3c557fcc51.webp)

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670158035680-bceb167a-e2cc-4de1-aeaa-6e7e2842a568.webp)

## 3. 注意事项

1. 编译时提示：Verification error in 和Had a hard failure verifying all classes, and was asked to abort in such situations.  
    **原因**：apk在预制进系统时，会对apk进行解析，并生成一个odex文件加快app的启动和运行。在高版本的Android系统中集成面向低版本开发的APP（一般低很多会报错），odex文件解析出错就会出现这样的提示。  
    **解决办法**：在放置apk的文件夹下的Android.mk中关闭预编译

```bash
# 关闭预编译，不会生成OAT文件
LOCAL_DEX_PREOPT := false
```

  
  
作者：林栩  
链接：[https://www.jianshu.com/p/c266492f6688](https://www.jianshu.com/p/c266492f6688)
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。