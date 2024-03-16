---
author: zjmantou
title: Android Systrace 14 流畅性实战 2 ：案例分析 - MIUI 桌面滑动卡顿分析
time: 2024-03-16 周六
tags:
  - Android
  - 资料
  - 性能优化
  - Systrace
  - Tools
---
不同的人对流畅性(卡顿掉帧)有不同的理解，对卡顿阈值也有不同的感知，所以有必要在开始这个系列文章之前，先把涉及到的内容说清楚，防止出现不同的理解，也方便大家带着问题去看这几篇问题，下面是一些基本的说明

1. 对手机用户来说，卡顿包含了很多场景，比如在 **滑动列表的时候掉帧**、**应用启动白屏过长**、**点击电源键亮屏慢**、**界面操作没有反应然后闪退**、**点击图标没有响应**、**窗口动画不连贯、滑动不跟手、重启手机进入桌面卡顿** 等场景，这些场景跟我们开发人员所理解的卡顿还有点不一样，开发人员会更加细分去分析这些问题，这是开发人员和用户之间的一个认知差异，这一点在处理用户(或者测试人员)的问题反馈的时候尤其需要注意
2. 对开发人员来说，上面的场景包括了 **流畅度**（滑动列表的时候掉帧、窗口动画不连贯、重启手机进入桌面卡顿）、**响应速度**（应用启动白屏过长、点击电源键亮屏慢、滑动不跟手）、**稳定性**（界面操作没有反应然后闪退、点击图标没有响应）这三个大的分类。之所以这么分类，是因为每一种分类都有不太一样的分析方法和步骤，快速分辨问题是属于哪一类很重要
3. 在技术上来说，**流畅度、响应速度、稳定性**（ANR）这三类之所以用户感知都是卡顿，是因为这三类问题产生的原理是一致的，都是由于主线程的 Message 在执行任务的时候超时，根据不同的超时阈值来进行划分而已，所以要理解这些问题，需要对系统的一些基本的运行机制有一定的了解，本文会介绍一些基本的运行机制
4. 流畅性这个系列主要是分析流畅度相关的问题，响应速度和稳定性会有专门的文章介绍，在理解了流畅性相关的内容之后，再去分析响应速度和稳定性问题会事半功倍
5. 流畅性这个系列主要是讲如何使用 Systrace (Perfetto) 工具去分析，之所以 Systrace 为切入点，是因为影响流畅度的因素很多，有 App 自身的原因、也有系统的原因。而 Systrace(Perfetto) 工具可以从一个整机运行的角度来展示问题发生的过程，方便我们去初步定位问题

Systrace (Perfetto) 工具的基本使用如果还不是很熟悉，那么需要优先去补一下上面列出的 [Systrace 基础知识系列](https://www.androidperformance.com/2019/05/26/Android_Systrace_0/)，本文假设你已经熟悉 Systrace(Perfetto)的使用了

# Systrace 分析卡顿问题的套路

使用 Systrace 分析卡顿问题，我们一般的流程如下

1. **复现卡顿的场景，抓取 Systrace，可以用 shell 或者手机自带的工具来抓取**
    
2. **双击抓出来的 trace.html 直接在 Chrome 中打开 Systrace 文件**
    
    1. 如果不能直接打开，可以在 Chrome 中输入 chrome://tracing/，然后把 Systrace 文件拖到里面就可以打开
    2. 或者使用 Perfetto View 中的 Open With Legacy UI 打开
3. **分析卡顿问前，我们需要了解问题发生的背景，以提高分析 Systrace 的效率**
    
    1. 用户（或者测试）的操作流程
    2. 卡顿复现概率
    3. 竞品机器是否也有同样的卡顿问题
4. **分析问题之前或者分析的过程中，也可以通过检查 Systrace 来了解一些基本的信息**
    
    1. CPU 频率、架构、Boost 信息等
    2. 是否触发温控：表现为cpu 频率被压低
    3. 是否是高负载场景：表现为 cpu 区域任务非常满
    4. 是否是低内存场景：表现为 lmkd 进程繁忙，App 进程的 HeapTaskDeamon 耗时，有很多的 Block io
5. **定位 App 进程在 Systrace 中的位置**
    
    1. **打开 Systrace 后，首先要首先要看的就是 App 进程，主要是 App 的主线程和渲染线程**，找到 Systrace 中每一帧耗时的部分，比如下面这种，可以看到 App 的 UI Thread 的红框部分，耗时 110ms，明显是不正常的（这个案例是 Bilibili 列表滑动卡顿）![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161724491.png)

        
    2. 事实上，所有超过一个 Vsync 周期的 doFrame 耗时（即大家看到的黄帧和红帧），我们都需要去看一下是否真的发生的掉帧，就算没有掉帧，也要看一下原因，比如下面这个![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161725667.png)

        
    3. Vsync 周期与刷新率的对照
        
        1. 60fps 对应的 Vsync 周期是 16.6ms
        2. 90fps 对应的 Vsync 周期是 11.1ms
        3. 120fps 对应的 Vsync 周期是 8.3ms
6. **分析 SurfaceFlinger 进程的主线程和 Binder 线程**
    
    1. 由于多个 Buffer 缓冲机制存在，App 主线程和渲染线程，有时候即使超过一个 Vsync 周期，也不一定会出现卡顿，所以这里我们需要看SurfaceFlinger 进程的主线程，来确认是否真的发生了卡顿 ，比如上面 3.1 部分的图，App 主线程耗时 110 ms，对应的 SurfaceFlinger 主线程如下![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161725364.png)

        
    2. Systrace 中的 SurfaceFlinger 进程区域，对应的 App 的 Buffer 个数也是空的![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161726656.png)

        
7. **从整机角度分析和 Binder 调用分析（不涉及可以不用看）**
    
    1. 上面的案例，可以很容易就看到是 App 自身执行耗时，那么只需要把耗时的部分涉及到的 View 找到，进行代码或者设计方面的优化就可以了
    2. 有时候 App 进程的主线程会出现大量的 Runnable 或者 Binder 调用耗时，也会导致 App 出现卡顿，这时候就需要分析整机问题，要看具体是什么原因导致大量的 Runnable 或者 Binder 调用耗时

按照这个流程分析之后，需要再反过来看各个进程，把各个线索联系起来，推断最有可能的原因

App 掉帧的原因非常多，有 APP 本身的问题，有系统原因导致卡顿的，也有硬件层的、整机卡的，这个可以参考下面四篇文章，不同的卡顿原因在 Systrace 中会有不同的表现

1. [Android 中的卡顿丢帧原因概述 - 方法论](https://www.androidperformance.com/2019/09/05/Android-Jank-Debug/)
2. [Android 中的卡顿丢帧原因概述 - 系统篇](https://www.androidperformance.com/2019/09/05/Android-Jank-Due-To-System/)
3. [Android 中的卡顿丢帧原因概述 - 应用篇](https://www.androidperformance.com/2019/09/05/Android-Jank-Due-To-App/)
4. [Android 中的卡顿丢帧原因概述 - 低内存篇](https://www.androidperformance.com/2019/09/18/Android-Jank-Due-To-Low-Memory/)

# 使用 Systrace 分析卡顿问题的案例

Systrace 作为分析卡顿问题的第一手工具，给开发者提供了一个从手机全局角度去看问题的方式，通过 Systrace 工具进行分析，我们可以大致确定卡顿问题的原因：是系统导致的还是应用自身的问题

当然 Systrace 作为一个工具，再进行深入的分析的时候就会有点力不从心，需要配合 TraceView + 源码来进一步定位和解决问题，最后再使用 Systrace 进行验证

所以本文更多地是讲如何发现和分析卡顿问题，至于如何解决，就需要后续自己寻找合适的解决方案了，比如对比竞品的 Systrace 表现、优化代码逻辑、优化系统调度、优化布局等

# 案例说明

个人在使用小米 10 Pro 的时候，在桌面滑动这个最常用的场景里面，总会有一种卡顿的感觉，10 Pro 是 90Hz 的屏幕，FPS 也是 90，所以一旦出现卡顿，就会有很明显的感觉(个人对这个也比较敏感)。之前没怎么关注，在升级 12.5 之后，这个问题还是没有解决，所以我想看看到底是怎么回事

抓了 Systrace 之后分析发现，这个卡顿场景是一个非常好的案例，所以把这个例子拿出来作为流畅度的一个实战分享

建议大家下载附件中的 Systrace，对照文章食用最佳

>1. 鉴于卡顿问题的影响因素比较多，所以在开始之前，我把本次分析所涉及的硬件、软件版本沟通清楚，如果后续此场景有优化，此文章也不会进行修改，以文章附件中的 Systrace 为准
>2. 硬件：小米 10 Pro
>3. 软件：MIUI 12.5.3 稳定版
>4. 小米桌面版本：RELEASE-4.21.11.2922-03151646

## 从 Input 事件开始[](https://www.androidperformance.com/2021/04/24/android-systrace-smooth-in-action-2/#%E4%BB%8E-Input-%E4%BA%8B%E4%BB%B6%E5%BC%80%E5%A7%8B)

这次抓的 Systrace 我只滑动了一次，所以比较好定位，滑动的 input 事件由一个 Input Down 事件 + 若干个 Input Move 事件 + 一个 Input Up 事件组成

在 Systrace 中，SystemServer 中的 InputDispatcher 和 InputReader 线程都有体现，我们这里主要看在 App 主线程中的体现

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161727186.png)


如上图，App 主线程上的 deliverInputEvent 标识了应用处理 input 事件的过程，input up 之后，就进入了 Fling 阶段，这部分的基础知识可以查看下面这两篇文章

1. [https://www.androidperformance.com/2019/11/04/Android-Systrace-Input/](https://www.androidperformance.com/2019/11/04/Android-Systrace-Input/)
2. [https://www.androidperformance.com/2020/08/20/weibo-imageload-opt-on-huawei/](https://www.androidperformance.com/2020/08/20/weibo-imageload-opt-on-huawei/)

## 分析主线程[](https://www.androidperformance.com/2021/04/24/android-systrace-smooth-in-action-2/#%E5%88%86%E6%9E%90%E4%B8%BB%E7%BA%BF%E7%A8%8B)

由于这次卡顿主要是松手之后才出现的，所以我们主要看 Input Up 之后的这段

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161727195.png)


主线程上面的 Frame 有颜色进行标注，一般有绿、黄、红三种颜色，上面的 Systrace 里面，没有红色的帧，只有绿色和黄色。那么黄色就一定是卡顿么？红色就一定是卡顿么？其实不一定，单单通过主线程，我们并不能确定是否卡顿，这个在下面会讲

从主线程我们没法确定是否发生了卡顿，我们找出了三个可疑的点，接下来我们看一下 RenderThread

## 分析渲染线程[](https://www.androidperformance.com/2021/04/24/android-systrace-smooth-in-action-2/#%E5%88%86%E6%9E%90%E6%B8%B2%E6%9F%93%E7%BA%BF%E7%A8%8B)

放大第一个可疑点，可以看到，这一帧总耗时在 19ms， RenderThread 耗时 16ms，且 RenderThread 的 cpu 状态都是 running（绿色），那么这一帧这么耗时的原因大概率是下面两个原因导致的：

1. RenderThread 本身耗时，任务比较繁忙
2. RenderThread 的任务受 CPU 影响（可能是频率低了、或者是跑到小核了）![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161727047.png)

由于只是可疑点，所以我们先不去看 cpu 相关的，先查看 SurfaceFlinger 进程，确定这里有卡顿发生

## 分析 SurfaceFlinger[](https://www.androidperformance.com/2021/04/24/android-systrace-smooth-in-action-2/#%E5%88%86%E6%9E%90-SurfaceFlinger)

对于 Systrace 中 SurfaceFlinger 部分解读不熟悉的可以先预习一下这篇文章 [https://www.androidperformance.com/2020/02/14/Android-Systrace-SurfaceFlinger/](https://www.androidperformance.com/2020/02/14/Android-Systrace-SurfaceFlinger/)

这里我们主要看两个点

1. App 对应的 BufferQueue 的 Buffer 情况。通过这个我们可以知道在 SurfaceFlinger 端，App 是否有可用的 Buffer 提供给 SurfaceFlinger 进行合成
2. SurfaceFlinger 主线程的合成情况。通过查看 SurfaceFlinger 在 sf-vsync 到来的时候是否进行了合成工作，就可以判断这一帧是否出现了卡顿。

**判断是否卡顿的标准如下**

1. 如果 SurfaceFlinger 主线程没有合成任务，而且 App 在这一个 Vsync 周期(vsync-app)进行了正常的工作，**但是对应的 App 的 BufferQueue 里面没有可用的 Buffer，那么说明这一帧卡了 — 卡顿出现**  
    这种情况如下图所示（也是上图中第一个疑点所在的位置）![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161728448.png)

    
2. 如果 SurfaceFlinger 进行了合成，而且 App 在这一个 Vsync 周期(vsync-app)进行了正常的工作，**但是对应的 App 的 BufferQueue 里面没有可用的 Buffer，那么这一帧也是卡了**，之所以 SurfaceFlinger 会正常合成，是因为有其他的 App 提供了可用来合成的 Buffer **— 卡顿出现**  
    这种情况如下图所示（也在附件的 Systrace 里面）  ![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161729923.png)

    
3. 如果 SurfaceFlinger 进行了合成，而且 App 在这一个 Vsync 周期(vsync-app)进行了正常的工作，而且对应的 App 的 BufferQueue 里面有可用的 Buffer，那么这一帧就会正常合成，此时没有卡顿出现 **— 正常情况**  
    正常情况如下，作为对比还是贴上来方便大家对比  ![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161729609.png)

回到本例的第一个疑点的地方，我们通过 SurfaceFlinger 端的分析，发现这一帧确实是掉了，原因是 App 没有准备好可用的 Buffer 供 SurfaceFlinger 来合成，那么接下来就需要看为什么这一帧 App 没有可用的 Buffer 给到 SurfaceFlinger

## 回到渲染线程[](https://www.androidperformance.com/2021/04/24/android-systrace-smooth-in-action-2/#%E5%9B%9E%E5%88%B0%E6%B8%B2%E6%9F%93%E7%BA%BF%E7%A8%8B)

上面我们分析这一帧所对应的 MainThread + RenderThread 耗时在 19ms，且 RenderThread 耗时就在 16ms，那么我们来看 RenderThread 的情况

出现这种情况主要是有下面两个原因

1. RenderThread 本身耗时，任务比较繁忙
2. RenderThread 的任务受 CPU 影响（可能是频率低了、或者是跑到小核了）

但是桌面滑动这个场景，负载并不高，且松手之后并没有多余的操作，View 更新之类的，本身耗时比前一帧多了将近 3 倍，可以推断不是自身负载加重导致的耗时

那么就需要看此时的 RenderThread 的 cpu 情况：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161729985.png)


既然在 Running 情况，我们就去 CPU Info 区域查看这一段时间这个任务的调度情况

## 分析 CPU 区域的信息[](https://www.androidperformance.com/2021/04/24/android-systrace-smooth-in-action-2/#%E5%88%86%E6%9E%90-CPU-%E5%8C%BA%E5%9F%9F%E7%9A%84%E4%BF%A1%E6%81%AF)

查看 CPU （Kernel 区域，这部分的基础知识可以查看 [Android Systrace 基础知识 - CPU Info 解读](https://www.androidperformance.com/2019/12/21/Android-Systrace-CPU) 和 [Android Systrace 基础知识 – 分析 Systrace 预备知识](https://www.androidperformance.com/2019/07/23/Android-Systrace-Pre/)）这两篇文章

回到这个案例，我们可以看到 App 对应的 RenderThread 大部分跑在 cpu 2 和 cpu 0 上，也就是小核上（这个机型是高通骁龙 865，有四个小核+3 个大核+1 个超大核）

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161730278.png)


其此时对应的频率也已经达到了小核的最高频率（1.8Ghz）

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161730200.png)

且此时没有 cpu boost 介入

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161731541.png)

那么这里我们猜想，之所以这一帧 RenderThread 如此耗时，是因为小核就算跑满了，也没法在这么短的时间内完成任务

那么接下来要验证我们的猜想，需要进行下面两个步骤

1. 对比其他正常的帧，是否有跑在小核的。如果有且没有出现掉帧，那么说明我们的猜想是错误的
2. 对比其他几个异常的帧，看看掉帧的原因是否也是因为 RenderThread 任务跑到了小核导致的。如果不是，那么就需要做其他的假设猜想

在用同样的流程分析了后面几个掉帧之后，我们发现

1. 对比其他正常的帧，没有在小核跑的，包括掉帧后的下一帧，调度器马上把 RenderThread 摆到了大核，没有出现连续掉帧的情况
2. 对比其他几个异常的帧，**都是由于 RenderThread 跑到了小核，但是小核的性能不足导致 RenderThread 执行耗时，最终引起卡顿**

至此，这一次的卡顿分析我们就找到了原因：**RenderThread 掉到了小核**

至于 RenderThread 的任务为啥跑着跑着就掉到了小核，这个跟调度器是有关系的，大小核直接的调度跟任务的负载有关系，任务从大核掉到小核、或者从小核迁移到大核，调度器这边都是有参数和算法来控制的，所以后续的优化可能需要从这方面去入手

1. 调整大小核迁移的阈值参数或者修改调度器算法
2. 参考竞品表现，看看竞品在这个场景的性能指标，调度情况等，分析竞品可能使用的策略

# Triple Buffer 在这个场景发挥了什么作用？

在 [Triple-Buffer-的作用](https://www.androidperformance.com/2019/12/15/Android-Systrace-Triple-Buffer/#Triple-Buffer-%E7%9A%84%E4%BD%9C%E7%94%A8) 这篇文章，讲到了 Triple Buffer 几个作用

1. 缓解掉帧
2. 减少主线程和渲染线程等待时间
3. 降低 GPU 和 SurfaceFlinger 瓶颈

那么在桌面滑动卡顿这个案例里面，Triple Buffer 发挥了什么作用呢？结论是：有的场景没有发挥作用，反而有副作用，导致卡顿现象更明显，下面是分析流程

可以看文章中 Triple Buffer **缓解掉帧** 的原理：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161731632.png)


在分析小米桌面滑动卡顿这个案例的时候，我发现在有一个问题，小米桌面对应的 App 的 BufferQueue，有时候会出现可用 Buffer 从 2 →0 ，这相当于直接把一个 Buffer 给抛弃掉了，如下图所示

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161731443.png)

这样的话，如果在后续的桌面 Fling 过程中，又出现了一次 RenderThread 耗时，那么就会以卡顿的形式直接体现出来，这样也就失去了 Triple Buffer 的**缓解掉帧**的作用了

下图可以看到，由于丢弃了一个 Buffer，导致再一次出现 RenderThread 耗时的时候，表现依然是无 Buffer 可用，出现掉帧

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161732345.png)


仔细看前面这段丢弃 Buffer 的逻辑，也很容易想到，这里本身就已经丢了一帧了，还把这个耗时帧所对应的 Buffer 给丢弃了（也可能丢弃的是第二帧），不管是哪种情况，滑动时候的每一帧的内容都是计算好的（参考 List Fling 的计算过程），如果把其中一帧丢了，再加上本身 SurfaceFlinger 卡的那一下，卡顿感会非常明显

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161732186.png)


举个例子,以滑动为例，offset 指的是离屏幕一个左边的距离

1. 正常情况下，滑动的时候，offset 是：`2→4→6→8→10→12`
2. 掉了一帧的情况下，滑动的 Offset 是：`2→4→6→6→8→10→12` （假设 计算 8 的这一帧超时了，就会看到两个 6 ，这是掉了一帧的情况）
3. 像上图里面，如果直接扔掉了那个耗时的帧，就会出现下面这种 Offset：`2→4→6→6→10→12` ，直接从 6 跳到了 10，相当于卡了 1 次，步子扯大了一次，感官上会觉得**卡+跳跃**

# 系列文章

1. [Systrace 流畅性实战 1 ：了解卡顿原理](https://www.androidperformance.com/2021/04/24/android-systrace-smooth-in-action-1/)
2. [Systrace 流畅性实战 2 ：案例分析: MIUI 桌面滑动卡顿分析](https://www.androidperformance.com/2021/04/24/android-systrace-smooth-in-action-2/)
3. [Systrace 流畅性实战 3 ：卡顿分析过程中的一些疑问](https://www.androidperformance.com/2021/04/24/android-systrace-smooth-in-action-3/)

# 附件

附件已经上传到了 Github 上，可以自行下载：[https://github.com/Gracker/SystraceForBlog/tree/master/Android_Systrace_Smooth_In_Action](https://github.com/Gracker/SystraceForBlog/tree/master/Android_Systrace_Smooth_In_Action)

1. xiaomi_launcher.zip : 桌面滑动卡顿的 Systrace 文件，这次案例主要是分析这个 Systrace 文件
2. xiaomi_launcher_scroll_all_the_time.zip : 桌面一直按着滑动的 Systrace 文件
3. oppo_launcher_scroll.zip ：对比文件

# 原文链接

1. [Systrace 流畅性实战 2 ：案例分析: MIUI 桌面滑动卡顿分析](https://www.androidperformance.com/2021/04/24/android-systrace-smooth-in-action-2/)

