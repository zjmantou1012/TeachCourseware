---
author: zjmantou
title: AndroidStudio 优秀插件推荐
time: 2023-10-19 周四
tags:
  - Android
  - 插件
---
# 插件安装方法

整理自己使用过程中的 AndroidStudio 插件，推荐优秀常用及自己做备份

## 1. Alibaba Java Coding Guidelines

[插件地址](https://plugins.jetbrains.com/plugin/10046-alibaba-java-coding-guidelines/)

简述：阿里巴巴 Java代码规范插件

可以指出代码中不规范的地方，避开一些隐患及提升自己的代码风格。

还有很多功能，值得推荐。

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294224077-0f1b4559-0485-483f-a13e-60ddd7f3b4ef.png)

## 2. Android ButterKnife Injections (Support Kotlin)

[插件地址](https://plugins.jetbrains.com/plugin/12012-android-butterknife-injections-support-kotlin-/)

简述：国内玩家升级Android ButterKnife Zelezny（停更），使用方法一样，功能更全。

对于使用ButterKnife插件的用户来说，必备。选用layout布局文件后，一键生成控件变量。

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294264916-bc748a48-cf4a-4464-9bc6-ab3a9f59970f.png)

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294271293-84abbd91-58f7-4b6d-bae7-e841555eb109.png)

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294276152-3cfd5b6f-3d0e-4586-8e6f-93374ab1ec4e.png)

## 3. GsonFormat

[插件地址](https://plugins.jetbrains.com/plugin/7654-gsonformat/)

简述：JSON 文本秒转对象

后台数据返回的JSON，使用Gson解析，手敲对象怕弄错？参数很多敲心烦？输入JSON，秒转对象。

Json示例：[http://cdn.dianshihome.com/tvlive/apk/channel/3rd.json](http://cdn.dianshihome.com/tvlive/apk/channel/3rd.json)（电视家的直播源）

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294303195-01a54625-6130-41bf-b01b-f793d3ae1e94.png)

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294307580-013c7c82-8f1f-486d-b9c5-17d87f90734b.png)

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294311560-910e3e44-a1f7-466c-83bc-25dc7b5fe2ae.png)

## 4. CMake simple highlighter

[插件地址](https://plugins.jetbrains.com/plugin/10089-cmake-simple-highlighter/)

简介：CMakeLists高亮插件

对于经常写JNI的同学来说，就很方便了。（借官方图）

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294353632-f7835cfb-26bb-4fdd-ad87-c4ba3dfbc20d.png)

## 5. Android Parcelable code generator

[插件地址](https://plugins.jetbrains.com/plugin/7332-android-parcelable-code-generator/)

简介：一键序列化对象

借刚才GsonFormat转出来的对象，快速序列化。

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294371111-c135189f-e01a-4543-adab-540727b7f35a.png)

## 6. Translation

[插件地址](https://plugins.jetbrains.com/plugin/8579-translation/)

简介：AndroidStudio内翻译与自动替换

英语不好的同学强烈推荐，支持翻译与替换。

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294391966-2ae9e036-8692-442d-92e2-e39db703960e.png)

## 7. SequenceDiagram

[插件地址](https://plugins.jetbrains.com/plugin/index?xmlId=SequenceDiagram)

简介：根据代码调用链路自动生成时序图，一键生成，便于整理思绪。

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294463131-f1e6365a-46d1-4b81-b4c4-592854e79eab.png)

## 8. simpleUMLCE

[插件地址](https://plugins.jetbrains.com/plugin/4946-simpleumlce/)

简介：根据代码自动生成UML的插件

自动生成UML来看源码，直观的继承，抽象结构可以很方便的让我们从架构角度看代码。

图上是自己工程中的一个Module，由于大部分在app中引用，所以内部线并不多。

（顺便赞一下Jetbrains的兼容性 ，2010年的插件在AndroidStudio 3.5.2上还能正常使用）

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294494046-d2b6b7a5-8274-4a73-883d-f96bf84305cd.png)

## 9. QAPlug全家桶

包含：QAPlug(主体)、QAPlug-Checkstyle(代码风格)、QAPlug-FindBugs（静态代码检查）、QAPlug-PMD(代码分析工具)

[QAPlug插件地址](https://plugins.jetbrains.com/plugin/4594-qaplug/)  
[QAPlug - Checkstyle插件地址](https://plugins.jetbrains.com/plugin/4595-qaplug--checkstyle/)  
[QAPlug - FindBugs插件地址](https://plugins.jetbrains.com/plugin/4597-qaplug--findbugs/)  
[QAPlug - PMD插件地址](https://plugins.jetbrains.com/plugin/4596-qaplug--pmd/)

简介：静态分析代码中潜在的问题、无用变量、空catch等各种问题，一键扫描。

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294519424-a2db7eea-84c2-4792-916b-09e136b58f02.png)

## 10. Rainbow Brackets

[插件地址](https://plugins.jetbrains.com/plugin/10080-rainbow-brackets/)

简介：彩色括号，复杂嵌套使用快速定位代码范围。

(借官方这张骚括号图)

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294534966-5cd839d3-af39-45e6-839b-0e3edb029b99.png)

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294539976-5a1b1a94-1aac-47e5-9627-b82388b405ae.png)

## 11. Codota

[插件地址](https://plugins.jetbrains.com/plugin/7638-codota/)

简介：代码提示Plus++，支持JDK及知名第三方库的提示、搜索、函数使用简介。

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294554835-4d2c5d6d-fe94-418e-82fd-7c67ad01a03c.png)

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294560217-2f41f4d8-378f-4382-a70a-122fd8241f59.png)

## 12 . ADB Idea

[插件地址](https://plugins.jetbrains.com/plugin/7380-adb-idea/)

简介：ADB命令中一些常用的小功能

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294575266-2e57826f-ace5-4ca1-bf4e-53abe81c2194.png)

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294584400-869372cc-1293-40d7-82e3-f036628328cd.png)

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1681294590382-bb0e4040-9766-417b-a46f-3c859afbd510.png)

二、插件安装方法

AndroidStudio > File > Setting > Plugins > Marketplace，然后输入插件名称，Install，重启ide。

转载请注明原帖地址：[https://blog.csdn.net/hx7013/article/details/103305985](https://blog.csdn.net/hx7013/article/details/103305985)

————————————————

版权声明：本文为CSDN博主「x024」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：[https://blog.csdn.net/hx7013/article/details/103305985](https://blog.csdn.net/hx7013/article/details/103305985)