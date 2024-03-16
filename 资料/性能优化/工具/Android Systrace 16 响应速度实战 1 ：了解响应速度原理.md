---
author: zjmantou
title: Android Systrace 16 响应速度实战 1 ：了解响应速度原理
time: 2024-03-16 周六
tags:
  - Android
  - 资料
  - 性能优化
  - Systrace
  - Tools
---
在讨论 Android 性能问题的时候，**卡顿**、**响应速度**、**ANR** 这三个性能相关的知识点通常会放到一起来讲，因为引起卡顿、响应慢、ANR 的原因类似，只不过根据重要程度，被人为分成了卡顿、响应慢、ANR 三种，所以我们可以定义广义上的卡顿，包含了卡顿、响应慢和 ANR 三种，所以如果用户反馈说手机卡顿或者 App 卡顿，大部分情况下都是广义上的卡顿，需要搞清楚，到底出现了哪一种问题

如果是动画播放卡顿、列表滑动卡顿这种，我们一般定义为 **狭义的卡顿**，对应的英文描述我觉得应该是 **Jank**；如果是应用启动慢、亮灭屏慢、场景切换慢，我们一般定义为 **响应慢** ，对应的英文描述我觉得应该是 **Slow** ；如果是发生了 ANR，那就是 **应用无响应问题** 。三种情况所对应的分析方法和解决方法不太一样，所以需要分开来讲

另外在 App 或者厂商内部，**卡顿**、**响应速度**、**ANR** 这几个性能指标都是有单独的标准的，比如 **掉帧率**、**启动速度**、**ANR 率**等，所以针对这些性能问题的分析和优化能力，对开发者来说就非常重要了

**本文是响应速度系列的第一篇，主要是讲**响应速度**相关的理论知识，包括性能工程概述、响应速度涉及到的知识点、响应速度的分析方法和套路等**

关于卡顿的文章可以参考这一篇 [Systrace 流畅性实战 1 ：了解卡顿原理](https://www.androidperformance.com/2021/04/24/android-systrace-smooth-in-action-1/) ，ANR 的文章后续会介绍，本文主要是讲响应速度相关的基本原理

Systrace (Perfetto) 工具的基本使用如果还不是很熟悉，那么需要优先去补一下 [Systrace 基础知识系列](https://www.androidperformance.com/2019/05/26/Android_Systrace_0/)，本文假设你已经熟悉 Systrace(Perfetto)的使用了

# 性能工程

在介绍响应速度的原理之前，这里先放一段 <**性能之巅**> 这本书中对于性能的描述，具体来说就是方法论，非常贴合本文的主题，也强烈推荐各位搞性能优化的同学，把这本书作为手头常读的方法论书籍：

--- 

# 性能是充满挑战的

系统性能工程是一个充满挑战的领域，具体原因有很多，其中包括以下事实，系统性能是主观的、复杂的，而且常常是多问题并存的

## 性能是主观的[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-1/#%E6%80%A7%E8%83%BD%E6%98%AF%E4%B8%BB%E8%A7%82%E7%9A%84)

1. 技术学科往往是客观的，太多的业界人士审视问题非黑即白。在进行软件故障查找的时候，判断 bug 是否存在或 bug 是否修复就是这样。bug 的出现总是伴随着错误信息，错误信息通常容易解读，进而你就明白错误为什么会出现了
2. 与此不同，性能常常是主观性的。开始着手性能问题的时候，对问题是否存在的判断都有可能是模糊的，在问题被修复的时候也同样，被一个用户认为是“不好”的性能，另一个用户可能认为是“好”的

## 系统是复杂的[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-1/#%E7%B3%BB%E7%BB%9F%E6%98%AF%E5%A4%8D%E6%9D%82%E7%9A%84)

1. 除了主观性之外，性能工程作为一门充满了挑战的学科，除了因为系统的复杂性，还因为对于性能，我们常常缺少一个明确的分析起点。有时我们只是从猜测开始，比如，责怪网络，而性能分析必须对这是不是一个正确的方向做出判断
2. 性能问题可能出在子系统之间复杂的互联上，即便这些子系统隔离时表现得都很好。也可能由于连锁故障（**cascading failure**）出现性能问题，这指的是一个出现故障的组件会导致其他组件产生性能问题。要理解这些产生的问题，你必须理清组件之间的关系，还要了解它们是怎样协作的
3. 瓶颈往往是复杂的，还会以意想不到的方式互相联系。修复了一个问题可能只是把瓶颈推向了系统里的其他地方，导致系统的整体性能并没有得到期望的提升。
4. 除了系统的复杂性之外，生产环境负载的复杂特性也可能会导致性能问题。在实验室环境很难重现这类情况，或者只能间歇式地重现
5. 解决复杂的性能问题常常需要全局性的方法。整个系统——包括自身内部和外部的交互——都可能需要被调查研究。这项工作要求有非常广泛的技能，一般不太可能集中在一人身上，这促使性能工程成为一门多变的并且充满智力挑战的工作

## 可能有多个问题并存[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-1/#%E5%8F%AF%E8%83%BD%E6%9C%89%E5%A4%9A%E4%B8%AA%E9%97%AE%E9%A2%98%E5%B9%B6%E5%AD%98)

1. 找到一个性能问题点往往并不是问题本身，在复杂的软件中通常会有多个问题
2. 性能分析的又一个难点：真正的任务不是寻找问题，而是辨别问题或者说是辨别哪些问题是最重要的
3. 要做到这一点，性能分析必须量化（**quantify**）问题的重要程度。某些性能问题可能并不适用于你的工作负载或者只在非常小的程度上适用。理想情况下，你不仅要量化问题，还要估计每个问题修复后能带来的增速。当管理层审查工程或运维资源的开销缘由时，这类信息尤其有用。
4. 有一个指标非常适合用来量化性能，那就是 延时（**latency**）

– 以上几段内容摘录自 <**性能之巅**>

--- 

# 响应速度概述

**响应速度**是应用 App 性能的重要指标之一。响应慢通常表现为**点击效果延迟**、**操作等待**或**白屏时间长**等，主要场景包括：

- **应用启动场景，包括冷启动、热启动、温启动等**
- **界面跳转场景，包括应用内页面跳转、App 之间跳转**
- **其他非跳转的点击场景（开关、弹窗、长按、控件选择、单击、双击等）**
- **亮灭屏、开关机、解锁、人脸识别、拍照、视频加载等场景**

从原理上来说，响应速度场景往往是由一个 input 事件（以 Message 的形式给到需要处理的应用主线程）触发（比如点击、长按、电源键、指纹等），由一个或者多个 Message 的执行结束为结尾，而这些 Message 中一般都有关键的界面绘制相关的 Message 。衡量一个场景的响应速度，我们通常从事件触发开始计时，到应用处理完成计时结束，这一段时间就称为响应时间。

如下图所示，**响应速度的问题，通常就是这些 Message 的某个执行超过预期（主观），导致最终完成的时间长于用户期待的时间**

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161739244.png)


由于响应速度是一个比较主观的性能指标（而流畅度就是一个很精确的指标，掉一帧就是掉一帧），而且根据角色的不同，对这个性能指标的判定也不同，比如 Android 系统开发者和应用开发者以及测试同学，对 **应用冷启动** 的起点和终点就有不同的判定：

1. **系统开发者** 往往从 input 中断开始看，部分以应用第一帧为结束点（因为比较好计算），部分以应用加载完成为结束点（比较主观，除非结束点比较容易通过工具去判断），主要是以优化应用的整体性能为主，涉及到的方面就比较广，包括 input 事件传递、SystemServer、SurfaceFlinger、Kernel 、Launcher 等
2. **App 开发者** 一般从 Application 的 onCreate 或者 attachContext 开始看，大部分以界面完全加载或者用户可操作为结束点，因为是自己的应用，结束点在代码里面可以主动加，主要还是以优化应用自身的启动速度为主，市面上讲启动速度优化的，大部分是讲这部分
3. **测试同学** 则更多从用户的真实体验角度来看，以桌面点击应用图标且应用图标变色为第一帧，内容完全加载为结束点。测试过程一般使用 **高速相机 + 自动化**，通过**机械手**和**图形识别技术**，可以自动进行响应速度测试并抓取相关的测试数据

# 响应速度问题分析思路

## 分清起点和终点[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-1/#%E5%88%86%E6%B8%85%E8%B5%B7%E7%82%B9%E5%92%8C%E7%BB%88%E7%82%B9)

分析响应速度，最重要的是要找到**起点**和**终点**，上一节讲到，不同角色的开发者，对这个性能指标的判定起点和终点都不一样；而且这个指标有很主观的成分，所以在开始的时候，就要跟各方来确定好起点和终点，具体的数值标准，下面一些手段可以帮助大家来确定

1. **竞品分析**。一般来说，响应速度这个指标都会有一个对标的竞品，竞品手机或者竞品 App，相同的条件下，竞品手机或者竞品 App 从点击到响应花费了多少时间，可以作为一个标准
2. **对比前一个版本**。有时候系统进行大版本升级或者 App 进行版本迭代，那么上一个版本的数据就可以拿来作为标准进行对比

一般来说，起点都比较好确定，无非是一个点击事件或者一个自定义的触发事件；而终点的确定就比较麻烦，比如如何确定一个复杂的 App （比如淘宝）启动完成的时间点，用 Systrace 的第一帧或者 Log 输出的 Displayed 时间或者 onWindowFocusChange 回调的时间显然是不准确的。目前市面上使用高速相机 + 图像识别来做是一个比较主流的做法

## 响应速度常见问题[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-1/#%E5%93%8D%E5%BA%94%E9%80%9F%E5%BA%A6%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)

### Android 系统自身原因导致响应慢[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-1/#Android-%E7%B3%BB%E7%BB%9F%E8%87%AA%E8%BA%AB%E5%8E%9F%E5%9B%A0%E5%AF%BC%E8%87%B4%E5%93%8D%E5%BA%94%E6%85%A2)

下面这些列举的是 Android 系统自身的原因，与 Android 机器的性能有比较大的关系，性能越差，越容易出现响应速度问题。下面就列出了 Android 系统原因导致的 App 响应速度出现问题的原因，以及这个时候 App 端在 Systrace 中的表现

1. **CPU 频率不足**
    - **App 端的表现**：主线程处于 Running 状态，但是执行耗时变长
2. **CPU 大小核调度：关键任务跑到了小核**
    - **App 端的表现**：Systrace 看主线程处于 Running 状态，但是执行耗时变长
3. **SystemServer 繁忙，主要影响**
    1. 响应 App 主线程 Binder 调用处理耗时
        - **App 端的表现**：Systrace 看主线程处于 Sleep 状态，在等待 Binder 调用返回
    2. 应用启动过程逻辑处理耗时
        - **App 端的表现**：Systrace 看主线程处于 Sleep 状态，在等待 Binder 调用返回
4. **SurfaceFlinger 繁忙，主要影响应用的渲染线程的 dequeueBuffer、queueBuffer**
    - **App 端的表现**：Systrace 看应用渲染线程的 dequeueBuffer、queueBuffer 处于 Binder 等待状态
5. **[系统低内存](https://www.androidperformance.com/2019/09/18/Android-Jank-Due-To-Low-Memory/)，低内存的时候，很大概率出现下面几种情况，都会对 SystemServer 和应用有影响**
    1. 低内存的时候，有些应用会频繁被杀和启动，而应用启动时一个重操作，会占用 CPU 资源，导致前台 App 启动变慢
        - **App 端的表现**：Systrace 看应用主线程 Runnable 状态变多，Running 状态变少，整体函数执行耗时增加
    2. 低内存的时候，, 很容易触发各个进程的 GC , , 用于内存回收的 HeapTaskDeamon、kswapd0 出现非常频繁
        - **App 端的表现**：Systrace 看应用主线程 Runnable 状态变多，Running 状态变少，整体函数执行耗时增加
    3. 低内存会导致磁盘 IO 变多, 如果频繁进行磁盘 IO , 由于磁盘 IO 很慢, 那么主线程会有很多进程处于等 IO 的状态, 也就是我们经常看到的 Uninterruptible Sleep
        - **App 端的表现**：Systrace 看应用主线程 Uninterruptible Sleep 和 Uninterruptible Sleep - IO 状态变多，Running 状态变少，整体函数执行耗时增加
6. **系统触发温控频率被限制：由于温度过高，CPU 最高频率被限制**
    - **App 端的表现**：主线程处于 Running 状态，但是执行耗时变长
7. **整机 CPU 繁忙：可能有多个高负载进程同时在运行，或者有单个进程负载过高跑满了 CPU**
    - **App 端的表现**：从 Systrace 来看，CPU 区域的任务非常满，所有的核心上都有任务在执行，App 的主线程和渲染线程多处于 Runnable 状态，或者频繁在 Runnable 和 Running 之间切换

### 应用自身原因[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-1/#%E5%BA%94%E7%94%A8%E8%87%AA%E8%BA%AB%E5%8E%9F%E5%9B%A0)

应用自身原因主要是应用启动时候的组件初始化、View 初始化、数据初始化耗时等，具体包括：

1. Application.onCreate：应用自身的逻辑 + 三方 SDK 初始化耗时
2. Activity 的生命周期函数：onStart、onCreate、onResume 耗时
3. Services 的生命周期函数耗时
4. Broadcast 的 onReceive 耗时
5. ContentProvider 初始化耗时（注意已经被滥用）
6. 界面布局初始化：measure、layout、draw 等耗时
7. 渲染线程初始化：setSurface、queueBuffer、dequeueBuffer、Textureupload 等耗时
8. Activity 跳转：从 SplashActivity 到 MainActivity 耗时
9. 应用向主线程 post 的耗时 Message 耗时
10. 主线程或者渲染线程等待子线程数据更新耗时
11. 主线程或者渲染线程等待子进程程数据更新耗时
12. 主线程或者渲染线程等待网络数据更新耗时
13. 主线程或者渲染线程 binder 调用耗时
14. WebView 初始化耗时
15. 初次运行 JIT 耗时

## 响应速度问题分析套路（以 Systrace 为主）[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-1/#%E5%93%8D%E5%BA%94%E9%80%9F%E5%BA%A6%E9%97%AE%E9%A2%98%E5%88%86%E6%9E%90%E5%A5%97%E8%B7%AF%EF%BC%88%E4%BB%A5-Systrace-%E4%B8%BA%E4%B8%BB%EF%BC%89)

1. 确认前提条件(老化,数据量、下载等)、操作步骤、问题现象，本地复现
2. 需要明确测试标准
    1. 启动时间的起点是哪里
    2. 启动时机的终点是哪里
3. 抓取所需的日志信息（Systrace、常规 log 等）
4. 首先分析 **Systrace**，大概找出差异的点
    1. 首先查看应用耗时点，分析对比机差异，这里可以把应用启动阶段分成好几段来看，来对比分析是哪部分时间增加
        1. Application 创建
        2. Activity 创建
        3. 第一个 doFrame
        4. 后续内容加载
        5. 应用自己的 Message
    2. 分析应用耗时的点
        1. 是否某一段方法自身执行耗时比较久（Running 状态） –> **应用自身问题**
        2. 主线程是否有大段 Running 状态，但是底下没有任何堆栈 –> **应用自身问题，加 TraceTag 或者使用 TraceView 来找到对应的代码逻辑**
        3. 是否在等 Binder 耗时比较久（Sleep 状态） –> **检测 Binder 服务端，一般是 SystemServer**
        4. 是否在等待子线程返回数据（Sleep 状态） –> **应用自身问题，通过查看 wakeup 信息，来找到依赖的子线程**
        5. 是否在等待子进程返回数据（Sleep 状态） –> **应用自身问题，通过查看 wakeup 信息，来找到依赖的子进程或者其他进程(一般是 ContentProvider 所在的进程)**
        6. 是否有大量的 Runnable –> **系统问题，查看 CPU 部分，看看是否已经跑满**
        7. 是否有大量的 IO 等待（Uninterruptible Sleep | WakeKill - Block I/O） –> **检查系统是否已经[低内存](https://www.androidperformance.com/2019/09/18/Android-Jank-Due-To-Low-Memory/#%E5%BD%B1%E5%93%8D%E4%B8%BB%E7%BA%BF%E7%A8%8B-IO-%E6%93%8D%E4%BD%9C)**
        8. RenderThread 是否执行 dequeueBuffer 和 queueBuffer 耗时 –> **查看 SurfaceFlinger**
    3. 如果分析是系统的问题，则根据上面耗时的点，查看系统对应的部分，一般情况要优先查看系统是否异常，参考上面列出的的系统原因，主要看下面四个区域（Systrace）
        1. [Kernel 区域](https://www.androidperformance.com/2019/12/21/Android-Systrace-CPU/)
            1. 查看关键任务是否跑在了小核 –> **一般小核是 0-3（也有特例），如果启动时候的关键任务跑到了小核，执行速度也会变慢**
            2. 查看频率是否没有跑满 –> **表现是核心频率没有达到最大值，比如最大值是 2.8Ghz，但是只跑到了 1.8Ghz，那么可能是有问题的**
            3. 查看 CPU 使用率，是否已经跑满了 –> **表现是 CPU 区域八个核心上，任务和任务之间没有空隙**
            4. 查看是否[低内存](https://www.androidperformance.com/2019/09/18/Android-Jank-Due-To-Low-Memory/)
                1. **应用进程状态有大量的 Uninterruptible Sleep | WakeKill - Block I/O**
                2. **HeapTaskDeamon 任务执行频繁**
                3. **kswapd0 任务执行频繁**
        2. [SystemServer 进程区域](https://www.androidperformance.com/2019/06/29/Android-Systrace-SystemServer/)
            1. input 事件读取和分发是否有异常 –> **表现是 input 事件传递耗时，比较少见**
            2. binder 执行是否耗时 –> **表现是 SystemServer 对应的 Binder 执行代码逻辑耗时**
            3. binder 等 am、wm 锁是否耗时–> **表现是 SystemServer 对应的 Binder 都在等待锁，可以通过 wakeup 信息跟踪等锁情况，分析等锁是不是由于应用导致的**
            4. 是否有应用频繁启动或者被杀 –> **在 Systrace 中查看 startProcess，或者查看 Event Log**
        3. [SurfaceFlinger 进程区域](https://www.androidperformance.com/2020/02/14/Android-Systrace-SurfaceFlinger/)
            1. dequeueBuffer 和 queueBuffer 是否执行耗时 –> **表现是 SurfaceFlinger 的对应的 Binder 执行 dequeueBuffer 和 queueBuffer 耗时**
            2. 主线程是否执行耗时 –> **表现是 SurfaceFlinger 主线程耗时，可能是在执行其他的任务**
        4. Launcher 进程区域（冷热启动场景）
            1. Launcher 进程处理点击事件是否耗时 –> **表现在处理 input 事件耗时**
            2. Launcher 自身 pause 是否耗时 –> **表现在执行 onPause 耗时**
            3. Launcher 应用启动动画是否耗时或者卡顿 –> **表现在动画耗时或者卡顿**
5. 初步分析有怀疑的点之后
    1. 如果是系统的原因，首先需要看应用自身是否能规避，如果不能规避，则转给系统来处理
    2. 如果是应用自身的原因，可以使用 TraceView（AS 自带的 CPU Profiler）、Simple Perf 等继续查看更加详细的函数调用信息，也可以使用 [TraceFix 插件](https://github.com/Gracker/TraceFix)，插入更多的 TraceTag 之后，重新抓取 Systrace 来对比分析
6. 问题可能有很多个原因
    1. 首先要把影响最大的因素找出来优化，影响比较小的因素可以先忽略
    2. 有些问题需要系统的配合才能解决，这时候需要跟系统一起进行调优（比如各大 App 厂商就会有专门跟手机厂商打交道的，手机厂商会以 SDK 的形式，暴露部分系统接口给 App 来使用，比如 Oppo 、华为、Vivo 等）
    3. 有些问题影响很小或者无解，这时候需要跟测试同学沟通清楚
    4. 有些问题是重复问题或不同平台的相同，可以在 Bug 库中搜索是否有案例

本篇文章主要是一个响应速度基础知识方面的一个普及，其中涉及到大量的系统知识，不熟悉的同学可以跟着 [Systrace 基础知识系列](https://www.androidperformance.com/2019/05/26/Android_Systrace_0/) 过一下

# 系列文章

1. [Systrace 响应速度实战 1 ：了解响应速度原理](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-1/)
2. [Systrace 响应速度实战 2 ：响应速度实战分析-以启动速度为例](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-2/)
3. [Systrace 响应速度实战 3 ：响应速度延伸知识](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-3/)
4. [Systrace 基础知识系列-放个链接在这里方便大家直接点过去](https://www.androidperformance.com/2019/05/26/Android_Systrace_0/)

# 参考文章

1. [Android 应用启动全流程分析](https://www.jianshu.com/p/37370c1d17fc)
2. [探究 | App Startup 真的能减少启动耗时吗](https://juejin.cn/post/6907493155659055111)
3. [Jetpack 新成员，App Startup 一篇就懂](https://blog.csdn.net/guolin_blog/article/details/108026357)
4. [App Startup](https://developer.android.google.cn/topic/libraries/app-startup)
5. [Android App 启动优化全记录](https://www.androidperformance.com/2019/11/18/Android-App-Lunch-Optimize/)
6. [Android application profiling](https://android.googlesource.com/platform/system/extras/+/master/simpleperf/doc/android_application_profiling.md)

# 原文链接

1. [Systrace 响应速度实战 1 ：了解响应速度原理](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-1/)
