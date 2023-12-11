---
author: zjmantou
title: Android新版本功能
time: 2023-12-05 周二
tags:
  - Android
  - Android版本
---
# Android14

## 冻结缓存应用，增加杀进程的能力

内核层面使用了cgroup v2 freezer，无法被hook。 

## 应用启动速度更快

增加了缓存应用的最大数量的限制，从而减少了冷启动应用的次数。 可以根据设备的内存容量进行调整。 

## 减少内存占用 

改进 Android 运行时（ART）对 Android 用户体验，ART 包含了优化措施，将代码大小平均减少了 9.3%，而不会影响性能。

## 屏幕截图检查

新增截图后的ScreenCaptureCallback的API回调。 

## 系统全屏通知

Notification. Builder. setFullScreenIntent

## 精确闹钟权限更改

ACTION_REQUEST_SCHEDULE_EXACT_ALARM

## 提供了对照片和视频的部分访问权限

## 最小targetSdkVersion限制（23）

## 返回手势动画



