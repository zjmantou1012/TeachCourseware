---
author: zjmantou
title: Android Systrace 11 基础知识 - Triple Buffer 解读
time: 2024-03-16 周六
tags:
  - Android
  - 资料
  - 性能优化
  - Systrace
  - Tools
---
本文是 Systrace 系列文章的第十一篇，主要是对 Systrace 中的 Triple Buffer 进行简单介绍，简单介绍了如何在 Systrace 中判断卡顿情况的发生，进行初步的定位和分析，以及介绍 Triple Buffer 的引入对性能的影响

本系列的目的是通过 Systrace 这个工具，从另外一个角度来看待 Android 系统整体的运行，同时也从另外一个角度来对 Framework 进行学习。也许你看了很多讲 Framework 的文章，但是总是记不住代码，或者不清楚其运行的流程，也许从 Systrace 这个图形化的角度，你可以理解的更深入一些。

# 怎么定义掉帧？

Systrace 中可以看到应用的掉帧情况，我们经常看到说主线程超过 16.6 ms 就会掉帧，其实不然，这和我们这一篇文章讲到的 Triple Buffer 和一定的关系，一般来说，Systrace 中我们从 App 端和 SurfaceFlinger 端一起来判断掉帧情况

## App 端判断掉帧[](https://www.androidperformance.com/2019/12/15/Android-Systrace-Triple-Buffer#App-%E7%AB%AF%E5%88%A4%E6%96%AD%E6%8E%89%E5%B8%A7)

如果之前没有看过 Systrace 的话，仅仅从理论上来说，下面这个 Trace 中的应用是掉帧了，其主线程的绘制时间超过了 16.6ms ,但其实不一定，因为 BufferQueue 和 TripleBuffer 的存在，此时 BufferQueue 中可能还有上一帧或者上上一帧准备好的 Buffer，可以直接被 SurfaceFlinger 拿去做合成，当然也可能没有

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161703177.png)


**所以从 Systrace 的 App 端我们是无法直接判断是否掉帧的，需要从 Systrace 里面的 SurfaceFlinger 端去看**

## SurfaceFlinger 端判断掉帧

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161703711.png)


SurfaceFlinger 端可以看到 SurfaceFlinger 主线程和合成情况和应用对应的 BufferQueue 中 Buffer 的情况。如上图，就是一个掉帧的例子。App 没有及时渲染完成，且此时 BufferQueue 中也没有前几帧的 Buffer，所以这一帧 SurfaceFlinger 没有合成对应 App 的 Layer，在用户看来这里就掉了一帧

而在第一张图中我们说从 App 端无法看出是否掉帧，那张图对应的 SurfaceFlinger 的 Trace 如下, 可以看到由于有 Triple Buffer 的存在, SF 这里有之前 App 的 Buffer,所以尽管 App 测一帧超过了 16.6 ms, 但是 SF 这里依然有可用来合成的 Buffer, 所以没有掉帧

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161703225.png)

## 逻辑掉帧[](https://www.androidperformance.com/2019/12/15/Android-Systrace-Triple-Buffer#%E9%80%BB%E8%BE%91%E6%8E%89%E5%B8%A7)

上面的掉帧我们是从渲染这边来看的，这种掉帧在 Systrace 中可以很容易就发现；还存在一种掉帧情况叫**逻辑掉帧**

**逻辑掉帧**指的是由于应用自己的代码逻辑问题，导致画面更新的时候，不是以均匀或者物理曲线的方式，而是出现跳跃更新的情况，这种掉帧一般在 Systrace 上没法看出来，但是用户在使用的时候可以明显感觉到

举一个简单的例子，比如说列表滑动的时候，如果我们滑动松手后列表的每一帧前进步长是一个均匀变化的曲线，最后趋近于 0，这样就是完美的；但是如果出现这一帧相比上一帧走了 20，下一帧相比这一帧走了 10，下下一帧相比下一帧走了 30，这种就是跳跃更新，在 Systrace 上每一帧都是及时渲染且 SurfaceFlinger 都及时合成的，但是用户用起来就是觉得会卡. 不过我列举的这个例子中，Android 已经针对这种情况做了优化，感兴趣的可以去看一下 android/view/animation/AnimationUtils.java 这个类，重点看下面三个方法的使用

```java
public static void lockAnimationClock(long vsyncMillis)  
public static void unlockAnimationClock()  
public static long currentAnimationTimeMillis()
```

Android 系统的动画一般不会有这个问题，但是应用开发者就保不齐会写这种代码，比如做动画的时候根据**当前的时间(而不是 Vsync 到来的时间)**来计算动画属性变化的情况，这种情况下，一旦出现掉帧，动画的变化就会变得不均匀，感兴趣的可以自己思考一下这一块

另外 Android 出现掉帧情况的原因非常多，各位可以参考下面三篇文章食用：

1. [Android 中的卡顿丢帧原因概述 - 方法论](https://www.androidperformance.com/2019/09/05/Android-Jank-Debug/)
2. [Android 中的卡顿丢帧原因概述 - 系统篇](https://www.androidperformance.com/2019/09/05/Android-Jank-Due-To-System/)
3. [Android 中的卡顿丢帧原因概述 - 应用篇](https://www.androidperformance.com/2019/09/05/Android-Jank-Due-To-App/)

# BufferQueue 和 Triple Buffer

## BufferQueue[](https://www.androidperformance.com/2019/12/15/Android-Systrace-Triple-Buffer#BufferQueue)

首先看一下 BufferQueue，BufferQueue 是一个生产者(Producer)-消费者(Consumer)模型中的数据结构，一般来说，消费者(Consumer) 创建 BufferQueue，而生产者(Producer) 一般不和 BufferQueue 在同一个进程里面

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161704605.png)


其运行逻辑如下

1. 当生产者(Producer) 需要 Buffer 时，它通过调用 dequeueBuffer（）并指定 Buffer 的宽度，高度，像素格式和使用标志，从 BufferQueue 请求释放 Buffer
2. 生产者(Producer) 将填充缓冲区，并通过调用 queueBuffer（）将缓冲区返回到队列。
3. 消费者(Consumer) 使用 acquireBuffer（）获取 Buffer 并消费 Buffer 的内容
4. 使用完成后，消费者(Consumer)将通过调用 releaseBuffer（）将 Buffer 返回到队列

Android 通过 Vsync 机制来控制 Buffer 在 BufferQueue 中的流动时机，如果对 Vsync 机制不了解，可以参考下面这两篇文章，看完后你会有个大概的了解

1. [Systrace 基础知识 - Vsync 解读](https://www.androidperformance.com/2019/12/01/Android-Systrace-Vsync/)
2. [Android 基于 Choreographer 的渲染机制详解](https://www.androidperformance.com/2019/10/22/Android-Choreographer/)

上面的流程比较抽象，这里举一个具体的例子，方便大家理解上面那张图，对后续了解 Systrace 中的 BufferQueue 也会有帮助。

在 Android App 的渲染流程里面，App 就是个生产者(Producer) ，而 SurfaceFlinger 是一个消费者(Consumer)，所以上面的流程就可以翻译为

1. 当 **App** 需要 Buffer 时，它通过调用 dequeueBuffer（）并指定 Buffer 的宽度，高度，像素格式和使用标志，从 BufferQueue 请求释放 Buffer
2. **App** 可以用 cpu 进行渲染也可以调用用 gpu 来进行渲染，渲染完成后，通过调用 queueBuffer（）将缓冲区返回到 App 对应的 BufferQueue(如果是 gpu 渲染的话，这里还有个 gpu 处理的过程)
3. **SurfaceFlinger** 在收到 Vsync 信号之后，开始准备合成，使用 acquireBuffer（）获取 App 对应的 BufferQueue 中的 Buffer 并进行合成操作
4. 合成结束后，**SurfaceFlinger** 将通过调用 releaseBuffer（）将 Buffer 返回到 App 对应的 BufferQueue

理解了 BufferQueue 的作用后，接下来来讲解一下 BufferQueue 中的 Buffer

从上面的图可以看到，BufferQueue 中的生产者和消费者通过 dequeueBuffer、queueBuffer、acquireBuffer、releaseBuffer 来申请或者释放 Buffer，那么 BufferQueue 中需要几个 Buffer 来进行运转呢？下面从单 Buffer，双 Buffer 和 Triple Buffer 的角度分析(注意这里只是从 Buffer 的角度来做分析的, 比如 App 测涉及到 Buffer 的是 RenderThread , 不过由于 RenderThread 与 MainThread 有一定的联系, 比如 unBlockUiThread 执行的时机, MainThread 也会因为 RenderThread 执行慢而被 Block 住)

## Single Buffer[](https://www.androidperformance.com/2019/12/15/Android-Systrace-Triple-Buffer#Single-Buffer)

单 Buffer 的情况下，因为只有一个 Buffer 可用，那么这个 Buffer 既要用来做合成显示，又要被应用拿去做渲染

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161704521.png)


理想情况下，单 Buffer 是可以完成任务的（有 Vsync-Offset 存在的情况下）

1. App 收到 Vsync 信号，获取 Buffer 开始渲染
2. 间隔 Vsync-Offset 时间后，SurfaceFlinger 收到 Vsync 信号，开始合成
3. 屏幕刷新，我们看到合成后的画面

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161704142.png)

但是很不幸，理想情况我们也就想一想，这期间如果 App 渲染或者 SurfaceFlinger 合成在屏幕显示刷新之前还没有完成，那么屏幕刷新的时候，拿到的 Buffer 就是不完整的，在用户看来，就有种撕裂的感觉

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161705412.png)

当然 Single Buffer 已经没有在使用，上面只是一个例子

## Double Buffer[](https://www.androidperformance.com/2019/12/15/Android-Systrace-Triple-Buffer#Double-Buffer)

Double Buffer 相当于 BufferQueue 中有两个 Buffer 可供轮转，消费者在消费 Buffer的同时，生产者也可以拿到备用的 Buffer 进行生产操作

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161705388.png)


下面我们来看理想情况下，Double Buffer 的工作流程

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161706480.png)

但是 Double Buffer 也会存在性能上的问题，比如下面的情况，App 连续两帧生产都超过 Vsync 周期(准确的说是错过 SurfaceFlinger 的合成时机) ，就会出现掉帧情况

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161706552.png)


## Triple Buffer[](https://www.androidperformance.com/2019/12/15/Android-Systrace-Triple-Buffer#Triple-Buffer)

Triple Buffer 中，我们又加入了一个 BackBuffer ，这样的话 BufferQueue 里面就有三个 Buffer 可以轮转了，当 FrontBuffer 在被使用的时候，App 有两个空闲的 Buffer 可以拿去生产，就算生产过程中有 GPU 超时，CPU 任然可以拿到新的 Buffer 进行生产(**即 SurfaceFling 消费 FrontBuffer，GPU 使用一个 BackBuffer，CPU使用一个 BackBuffer**)

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161706144.png)

下面就是引入 Triple Buffer 之后，解决了 Double Buffer 中遇到的由于 Buffer 不足引起的掉帧问题

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161706331.png)

这里把两个图放到一起来看，方便大家做对比（一个是 Double Buffer 掉帧两次，一个是使用 Triple Buffer 只掉了一帧）

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161707152.png)


# Triple Buffer 的作用

## 缓解掉帧[](https://www.androidperformance.com/2019/12/15/Android-Systrace-Triple-Buffer#%E7%BC%93%E8%A7%A3%E6%8E%89%E5%B8%A7)

从上一节 Double Buffer 和 Triple Buffer 的对比图可以看到，在这种情况下（出现连续主线程超时），三个 Buffer 的轮转有助于缓解掉帧出现的次数（从掉帧两次 -> 只掉帧一次）

所以从第一节如何定义掉帧这里我们就知道，App 主线程超时不一定会导致掉帧，由于 Triple Buffer 的存在，部分 App 端的掉帧(主要是由于 GPU 导致)，到 SurfaceFlinger 这里未必是掉帧，这是看 Systrace 的时候需要注意的一个点

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161707972.png)


## 减少主线程和渲染线程等待时间[](https://www.androidperformance.com/2019/12/15/Android-Systrace-Triple-Buffer#%E5%87%8F%E5%B0%91%E4%B8%BB%E7%BA%BF%E7%A8%8B%E5%92%8C%E6%B8%B2%E6%9F%93%E7%BA%BF%E7%A8%8B%E7%AD%89%E5%BE%85%E6%97%B6%E9%97%B4)

**双 Buffer 的轮转**， App 主线程有时候必须要等待 SurfaceFlinger(消费者)释放 Buffer 后，才能获取 Buffer 进行生产，这时候就有个问题，现在大部分手机 SurfaceFlinger 和 App 同时收到 Vsync 信号，如果出现App 主线程等待 SurfaceFlinger(消费者)释放 Buffer ，那么势必会让 App 主线程的执行时间延后，比如下面这张图，可以明显看到：**Buffer B 并不是在 Vsync 信号来的时候开始被消费(因为还在使用)，而是等 Buffer A 被消费后，Buffer B 被释放，App 才能拿到 Buffer B 进行生产，这期间就有一定的延迟，会让主线程可用的时间变短**

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161707196.png)


我们来看一下在 Systrace 中的上面这种情况发生的时候的表现

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161707088.png)


而 三个 Buffer 轮转的情况下，则基本不会有这种情况的发生，渲染线程一般在 dequeueBuffer 的时候，都可以顺利拿到可用的 Buffer （当然如果 dequeueBuffer 本身耗时那就不是这里的讨论范围了）

## 降低 GPU 和 SurfaceFlinger 瓶颈[](https://www.androidperformance.com/2019/12/15/Android-Systrace-Triple-Buffer#%E9%99%8D%E4%BD%8E-GPU-%E5%92%8C-SurfaceFlinger-%E7%93%B6%E9%A2%88)

这个比较好理解，双 Buffer 的时候，App 生产的 Buffer 必须要及时拿去让 GPU 进行渲染，然后 SurfaceFlinger 才能进行合成，一旦 GPU 超时，就很容易出现 SurfaceFlinger 无法及时合成而导致掉帧

在三个 Buffer 轮转的时候，App 生产的 Buffer 可以及早进入 BufferQueue，让 GPU 去进行渲染（因为不需要等待，就算这里积累了 2 个 Buffer，下下一帧才去合成，这里也会提早进行，而不是在真正使用之前去匆忙让 GPU 去渲染），另外 SurfaceFlinger 本身的负载如果比较大，三个 Buffer 轮转也会有效降低 dequeueBuffer 的等待时间

比如下面两张图，就是对应的 SurfaceFlinger 和 App 的**双 Buffer 掉帧**情况，由于 SurfaceFlinger 本身就比较耗时（特定场景），而 App 的 dequeueBuffer 得不到及时的响应，导致发生了比较严重的掉帧情况。在换成 Triple Buffer 之后，这种情况就基本上没有了

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161708041.png)


# Debug Triple Buffer

## Dumpsys SurfaceFlinger[](https://www.androidperformance.com/2019/12/15/Android-Systrace-Triple-Buffer#Dumpsys-SurfaceFlinger)

dumpsys SurfaceFlinger 可以查看 SurfaceFlinger 输出的众多当前的状态，比如一些性能指标、Buffer 状态、图层信息等，后续有篇幅的话可以单独拿出来讲，下面是截取的 Double Buffer 情况下和 Triple Buffer 情况下的各个 App 的 Buffer 使用情况，可以看到不同的 App，在负载不一样的情况下，对 Triple Buffer 的使用率是不一样的；Double Buffer 则完全使用的是双 Buffer

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161709236.png)


## 关闭 Triple Buffer[](https://www.androidperformance.com/2019/12/15/Android-Systrace-Triple-Buffer#%E5%85%B3%E9%97%AD-Triple-Buffer)

不同 Android 版本属性设置不一样(这是 Google 的一个逻辑 Bug，Android 10 上面已经修复了)

### Android 版本 <= Android P

```c
//控制代码  
property_get("ro.sf.disable_triple_buffer", value, "1");  
mLayerTripleBufferingDisabled = atoi(value);  
ALOGI_IF(mLayerTripleBufferingDisabled, "Disabling Triple Buffering");
```

**修改对应的属性值，然后重启 Framework**

```shell
//按顺序执行下面的语句(需要 Root 权限)  
adb root  
adb shell setprop ro.sf.disable_triple_buffer 0  
adb shell stop && adb shell start
```

### Android 版本 > Android P

```c
/控制代码  
property_get("ro.sf.disable_triple_buffer", value, "0");  
mLayerTripleBufferingDisabled = atoi(value);  
ALOGI_IF(mLayerTripleBufferingDisabled, "Disabling Triple Buffering");
```

**修改对应的属性值，然后重启 Framework**

```shell
//按顺序执行下面的语句(需要 Root 权限)  
adb root  
adb shell setprop ro.sf.disable_triple_buffer 1  
adb shell stop && adb shell start
```

# 参考

1. [https://source.android.google.cn/devices/graphics](https://source.android.google.cn/devices/graphics)

# 附件

本文涉及到的附件也上传了，各位下载后解压，使用 **Chrome** 浏览器打开即可  
[点此链接下载文章所涉及到的 Systrace 附件](https://github.com/Gracker/SystraceForBlog/tree/master/Android_Systrace_TripleBuffer)


# 原文链接

1. [Systrace 基础知识 - Triple Buffer 解读](https://www.androidperformance.com/2019/12/15/Android-Systrace-Triple-Buffer)
