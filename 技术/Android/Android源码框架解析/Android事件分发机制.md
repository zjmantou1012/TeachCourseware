---
author: zjmantou
title: Android事件分发机制
time: 2023-11-23 周四
tags:
  - Android
  - 源码
---
https://zhuanlan.zhihu.com/p/635722243

- dispatchTouchEvent()：负责将事件分发给子视图或者自己处理。
- onInterceptTouchEvent()：只有在ViewGroup中才有，用于拦截事件，决定是否传递给子视图。
- onTouchEvent()：负责处理事件，返回true表示消费了事件，返回false表示不消费事件。




![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311232138780.png)
