---
author: zjmantou
title: Flutter与React Native表
time: 2023-09-15 周五
tags:
  - 跨平台
  - Flutter
  - React Native
---
|      | RN                                               | Flutter            |
| ---- | ------------------------------------------------ | ------------------ |
| 语言 | JS，开发者更多                                   | Dart，容易学习上手 |
| 性能 | 有一定优势，直接编译为ARM或X86                   | 有JS运行时         |
| 生态 | 更丰富                                           | 在完善             |
| UI   | 依赖原生组件渲染，不同平台可能需要编写不同UI代码 | UI一致性           |

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309151524403.png)

RN：IOS自带JSCore，Android还需要集成各种so库。  
Flutter：Android自带Skia库，ios没有。  

# 跨平台的意义
跨平台的市场优势不在于性能或学习成本，甚至平台适配会更耗费时间，但是它最终能让代码逻辑（特别是业务逻辑），无缝的复用在各个平台上，降低了重复代码的维护成本，保证了各平台间的统一性。  

# 搭建环境
RN：npm、node、react-native-cli  
flutter：flutterSDK、Android Studio/VSCode 的Dart与Flutter的插件

# 实现原理
都属于单页面应用，最大不同在于UI的构建




#### 参考资料
[全网最全 Flutter 与 React Native 深入对比分析](https://zhuanlan.zhihu.com/p/70070316)