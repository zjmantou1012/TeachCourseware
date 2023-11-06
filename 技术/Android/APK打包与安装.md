---
author: zjmantou
title: APK打包与安装
time: 2023-11-06 周一
tags:
  - Android
  - 技术
---
# 打包流程

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311062307326.png)

- 通过AAPT工具打包资源（包括AndroidManifest.xml、布局文件、各种xml资源等），生成R.java文件
- 通过AIDL工具处理AIDL文件，生成相应的Java文件
- 通过Java compiler（java编译器）编译R.java、Java接口文件、Java源文件，生成.class文件
- 通过dex命令将.class文件和第三方库的.class文件处理生成classes.dex文件，该过程主要完成Java字节码转换成Dalvik字节码，压缩常量池以及清除冗余信息等工作
- 通过apkBuilder工具将资源文件、dex文件打包生成apk文件
- 通过Jarsigner工具，利用KeyStore对生成的apk文件进行签名
- 如果是正式版的APK，还会利用ZipAlign工具进行对齐处理，对齐的过程就是将APK文件中所有的资源文件距离文件的起始距位置都偏移4字节的整数倍，这样通过内存映射访问APK文件的速度会更快，并且会减少其在设备上运行时的内存占用。



# 安装流程

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311062303189.png)

1. 复制apk到/data/app目录下，解压并扫描安装包
2. 资源管理器解析apk里面的资源文件
3. 解析AndroidManifes.xml文件，并在/data/data/目录下创建对应的应用数据目录
4. 然后对dex进行优化，并保存在dalvik-cache目录下
5. 将AndroidManifes.xml中解析出来的四大组件信息注册到PackageManagerService中
6. 安装完成，发送广播；

# apk组成

- dex：最终生成的Dalvik字节码
- res：存放资源文件的目录
- asserts：额外建立的资源文件夹
- lib：如果存在的话，存放的是ndk编出来的so库
- META-INF：存放签名信息

MANIFEST.MF（清单文件）：其中每一个资源文件都有一个SHA-256-Digest签名，MANIFEST.MF文件的SHA256（SHA1）并base64编码的结果即为CERT.SF中的SHA256-Digest-Manifest值。

CERT.SF（待签名文件）：除了开头处定义的SHA256（SHA1）-Digest-Manifest值，后面几项的值是对MANIFEST.MF文件中的每项再次SHA256并base64编码后的值。

CERT.RSA（签名结果文件）：其中包含了公钥、加密算法等信息。首先对前一步生成的MANIFEST.MF使用了SHA256（SHA1）-RSA算法，用开发者私钥签名，然后在安装时使用公钥解密。最后，将其与未加密的摘要信息（MANIFEST.MF文件）进行对比，如果相符，则表明内容没有被修改。

- androidManifest：程序的全局清单配置文件。
- resources.arsc：编译后的二进制资源文件。

#### [签名算法的原理](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F286d2b372334 "https://www.jianshu.com/p/286d2b372334")

