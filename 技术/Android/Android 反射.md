---
author: zjmantou
title: Android 反射
time: 2023-10-22 周日
tags:
  - Android
  - 技术
---
# 反射原理

# 反射的版本限制

Android 9.0中getDeclaredField()方法被禁用反射，在release版本中会抛NoMethodExeception错误，原因是
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310221941837.png)
红框中是true，直接返回null，然后就会抛出异常。
解决方法：
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310221943666.png)
第一个方法中的if条件判断返回为true。

## 参考链接
[Android 9.0+ 限制反射系统api原理与解决方案](https://www.jianshu.com/p/47686820c970)

## Android11禁用了元反射

## 总结
反射出现问题时 , 必须找到一个可以反射的反射点挂钩子 , 如在 A 位置无法进行反射 , 就在 B 位置挂 Hook 钩子 ;

