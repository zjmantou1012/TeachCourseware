---
author: zjmantou
title: adb常用命令笔记
time: 2024-08-05 周一
tags:
  - 笔记
  - Android
  - adb
---
# Monkey 

```shell
$ adb shell monkey [options] <event-count>
```

示例： 

```shell
$ adb shell monkey -p your.package.name -v 500
```

# 查看系统属性

```shell
adb shell getprop
```

# 内存 

```shell
$ adb shell
//表示堆内存增长到某个大小后才会触发 GC。
$ getprop dalvik.vm.heapgrowthlimit
//性值表示堆内存的最大允许大小。
$ getprop dalvik.vm.heapsize

```

