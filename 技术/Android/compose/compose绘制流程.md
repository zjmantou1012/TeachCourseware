---
author: zjmantou
title: compose绘制流程
time: 2023-11-08 周三
tags:
  - Android
  - Jetpack
  - Compose
---
compose函数最终通过ComposeView的setContent方法设置到ComposeView里面  

Compose渲染的时候，每一个组件都是一个LayoutNode，组成一个LayoutNode树。

Compose函数最终会调用到Layout方法，生成ComposeUiNode，LayoutNode是他的实现类；

UiApplier：LayoutNode树的管理器。

测量和布局都是Compose自己实现，最终调用了Canvas

