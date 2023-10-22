---
author: zjmantou
title: 设备导入HTTPS证书（不抓取HTTPS数据可跳过此步骤）
time: 2023-10-19 周四
tags:
  - Android
---
# 使用openssl生成系统证书（win系统）

一、首先将fiddler证书导出来保存为Fiddler.cer

![](https://cdn.nlark.com/yuque/0/2022/jpeg/26044650/1648993247380-ce7fba67-2d24-424e-8cf5-86525a040c51.jpeg)

二、安装OpenSSL

官方下载地址：[https://www.openssl.org/source/](https://www.openssl.org/source/)

OpenSSL官网没有提供windows版本的安装包，可以选择其他开源平台提供的工具。例如 [http://slproweb.com/products/Win32OpenSSL.html](http://slproweb.com/products/Win32OpenSSL.html)

工具下载：[Win64OpenSSL-1_1_1g.exe](https://miguconfluence.cmread.com/download/attachments/31413982/Win64OpenSSL-1_1_1g.exe?version=1&modificationDate=1595839311223&api=v2)

参考连接：[Windows安装使用Openssl](https://blog.csdn.net/qq_39081974/article/details/81059022)

三、Openssl安装完成后把证书导入到Linux系统下并进行转换

打开cmd，指定到fiddler.cer的目录

命令：

1、openssl x509 -inform der -in fiddler.cer -out fiddler.pem

2、openssl x509 -inform PEM -subject_hash_old -in fiddler.pem

![](https://cdn.nlark.com/yuque/0/2022/jpeg/26044650/1648993249281-f7a9ec58-e7b8-4eff-87b9-48588b0f3c13.jpeg)

得到类似字符串：269953fb

3、type fiddler.pem > 269953fb.0

4、openssl x509 -inform PEM -text -in fiddler.pem -out /dev/null >> 269953fb.0

注：该命令会报一行错，可以忽略

5、openssl x509 -inform PEM -subject_hash -in fiddler.pem

![](https://cdn.nlark.com/yuque/0/2022/jpeg/26044650/1648993249348-8b6fb4b1-fca9-4cae-bf52-7d29ebcb4631.jpeg)

得到类似字符串：035f9290

6、type fiddler.pem > 035f9290.0

7、openssl x509 -inform PEM -text -in fiddler.pem -out /dev/null >> 035f9290.0

四、把生成好的269953fb.0、 035f9290.0这两个文件放入手机/system/etc/security/cacerts 目录下，并修改269953fb.0、 035f9290.0这2个文件的读写权限：

修改读写权限命令：chmod 777 269953fb.0

![](https://cdn.nlark.com/yuque/0/2022/jpeg/26044650/1648993246739-80fa0e59-aaca-4264-b8df-0119c9f41daf.jpeg)

  
  

参考连接：[Fiddler https最新抓包方法(Android 9.0)](https://blog.csdn.net/liuluok123/article/details/95971731)

[  
](https://miguconfluence.cmread.com/pages/viewpage.action?pageId=31413982)

以下面两个证书文件为例：

[035f9290.0](https://miguconfluence.cmread.com/download/attachments/14374798/035f9290.0?version=1&modificationDate=1595837241000&api=v2)

[269953fb.0](https://miguconfluence.cmread.com/download/attachments/14374798/269953fb.0?version=1&modificationDate=1595837241000&api=v2)

1）adb push \269953fb.0 /system/etc/security/cacerts

2）adb push \035f9290.0 /system/etc/security/cacerts

3）adb root

4）adb remount

5）开启文件读写权限

adb shell

cd system/etc/security/cacerts

chmod 777 035f9290.0

chmod 777 269953fb.0

6）重启设备