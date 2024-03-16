---
author: zjmantou
title: Android Systrace 02 基础知识 -- 分析 Systrace 预备知识
time: 2024-03-15 周五
tags:
  - Android
  - 资料
  - 性能优化
  - Systrace
  - Tools
---
本文是 Systrace 系列文章的第三篇，解释一下为何大家总是强调 60 fps。60 fps 是一个软件的概念，与屏幕刷新率里面提到的 60hz 是不一样的，可以参考这篇文章：[新的流畅体验，90Hz 漫谈](https://www.androidperformance.com/2019/05/15/90hz-on-android/)

本系列的目的是通过 Systrace 这个工具，从另外一个角度来看待 Android 的运行，从另外一个角度来对 Framework 进行学习。也许你看了很多讲 Framework 的文章，但是总是记不住代码，或者不清楚其运行的流程，也许从 Systrace 这个图形化的角度，你可以理解的更深入一些。

# 正文

今天来讲一下为何我们讲到流畅度，要首先说 60 帧。

我们先来理一下基本的概念：

1. 60 fps 的意思是说，画面每秒更新 60 次
2. 这 60 次更新，是要均匀更新的，不是说一会快，一会慢，那样视觉上也会觉得不流畅
3. 每秒 60 次，也就是 1/60 ~= 16.67 ms 要更新一次

在理解了上面的基本概念之后，我们再回到 Android 这边，为何 Android 现在的渲染机制，是使用 60 fps 作为标准呢？这主要和屏幕的刷新率有关。

## 基本概念[](https://www.androidperformance.com/2019/05/27/why-60-fps/#%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5)

1. 我们前面说的 60 fps，是针对软件的，我们一般称为 fps
2. 屏幕的刷新率，是针对硬件的，现在大部分手机屏幕的刷新率，都维持在60 HZ，**移动设备上一般使用60HZ，是因为移动设备对于功耗的要求更高，提高手机屏幕的刷新率，对于手机来说，逻辑功耗会随着频率的增加而线性增大，同时更高的刷新率，意味着更短的TFT数据写入时间，对屏幕设计来说难度更大。**
3. 屏幕刷新率 60 HZ 只能说**够用**，在目前的情况下是最优解，但是未来肯定是高刷新率屏幕的天下（2023 年的现在 120Hz 已经是 Android 手机的标配了，连 iOS 都已经上到了 120Hz），个人觉得主要依赖下面几点的突破：
    4. 电池技术
    5. 软件技术
    6. 硬件能力

目前的情况下

1. 60 FPS 的情况下：Android 的渲染机制是 16.67 ms 绘制一次， 60hz 的屏幕也是 16.67 ms 刷新一次
2. 120 FPS 的情况下：Android 的渲染机制是 8.33 ms 绘制一次， 120hz 的屏幕也是 8.33 ms 刷新一次

## 效果提升[](https://www.androidperformance.com/2019/05/27/why-60-fps/#%E6%95%88%E6%9E%9C%E6%8F%90%E5%8D%87)

如果要提升，那么软件和硬件需要一起提升，光提升其中一个，是基本没有效果的，比如你屏幕刷新率是 75 hz，软件是 60 fps，每秒软件渲染60次，你刷新 75 次，是没有啥效果的，除了重复帧率费电；同样，如果你屏幕刷新率是 30 hz，软件是 60 fps，那么软件每秒绘制的60次有一半是没有显示就被抛弃了的。

如果你想体验120hz 刷新率的屏幕，建议你试试 ipad pro ，用过之后你会觉得，60 hz 的屏幕确实有改善的空间。

这一篇主要是简单介绍，如果你想更深入的去了解，可以去 Google 一下，另外 Google 出过一个短视频，介绍了 Why 60 fps， 有条件翻墙的同学可以去看看 ：

1. [Why 60 fps](https://www.youtube.com/watch?v=CaMTIgxCSqU)
2. [玩游戏为何要60帧才流畅，电影却只需24帧](https://www.youtube.com/watch?v=--OKrYxOb6Y)

下面这张图是 Android 应用在一帧内所需要完成的任务，后续我们还会详细讲这个：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161445432.png)





# 原文链接

[Systrace 基础知识 - Why 60 fps ？](https://www.androidperformance.com/2019/05/27/why-60-fps/)
