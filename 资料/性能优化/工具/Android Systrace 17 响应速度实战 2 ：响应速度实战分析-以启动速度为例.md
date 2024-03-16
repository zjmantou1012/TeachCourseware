---
author: zjmantou
title: Android Systrace 17 响应速度实战 2 ：响应速度实战分析-以启动速度为例
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

**本文是响应速度系列的第二篇，主要是以 Android App 冷启动为例，讲解如何使用 Systrace 来分析 App 冷启动**

Systrace (Perfetto) 工具的基本使用如果还不是很熟悉，那么需要优先去补一下 [Systrace 基础知识系列](https://www.androidperformance.com/2019/05/26/Android_Systrace_0/)，本文假设你已经熟悉 Systrace(Perfetto)的使用了

# 1. 准备工作

这个案例和对应的 Systrace 偏工程化一些，省略了很多细节，因为应用的启动流程涉及的知识非常广，如果每个都细化的话，会有很大的篇幅。推荐大家看这篇文章，非常详细：[Android 应用启动全流程分析](https://www.jianshu.com/p/37370c1d17fc)

所以这里以 Systrace 为主线，讲解应用启动的时候各个关键模块的大概工作流程。了解大概流程之后，就可以分段去深入自己感兴趣或者自己负责的部分，这里首先放一张 Systrace 和手机截图所对应的图，大家可以先看看这个图，然后再往下看（博客里面 Perfetto 和 Systrace 混合使用）

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161742546.png)

为了更方便分析应用冷启动，我们需要做下面的准备工作

1. 打开 Binder 调试，方便在 Trace 中显示 Binder 信息﻿（即可以在 Systrace 中看到 Binder 调用的函数）- 需要 Root
    1. 开启 ipc debug： `adb shell am trace-ipc start`
    2. 抓取结束后，可以执行下面的命令关闭﻿`adb shell am trace-ipc stop --dump-file /data/local/tmp/ipc-trace.txt`
2. Trace 命令加入 **irq** tag，默认的命令不包含 irq，需要自己加 irq 的 TAG,这样打开 Trace 之后，就可以看到 irq 相关的内容，最后的抓 trace 命令如下：﻿  
    ﻿ `python /mnt/d/Android/platform-tools/systrace/systrace.py gfx input view webview wm am sm rs bionic power pm ss database network adb idle pdx sched irq freq idle disk workq binder_driver binder_lock -a com.xxx.xxx` ,注意这里的 com.xxx.xxx 换成自己的包名，如果不是调试特定的包名，可以去掉 -a com.xxx.xxx
3. 推荐 ：如果要 Debug 的 App 可以进行编译（即可以使用 Gradle 编译，一般自己开发的项目都可以），可以在分析响应速度问题的时候，引入 TraceFix 库(接入方法参考 [https://github.com/Gracker/TraceFix)。接入之后，编译的时候就会进行代码插桩，在](https://github.com/Gracker/TraceFix)%E3%80%82%E6%8E%A5%E5%85%A5%E4%B9%8B%E5%90%8E%EF%BC%8C%E7%BC%96%E8%AF%91%E7%9A%84%E6%97%B6%E5%80%99%E5%B0%B1%E4%BC%9A%E8%BF%9B%E8%A1%8C%E4%BB%A3%E7%A0%81%E6%8F%92%E6%A1%A9%EF%BC%8C%E5%9C%A8) App 代码的每一个函数中都插入 Trace 点，这样在分析的时候可以看到更详细的 App 的信息
    1. 使用插件前，只能看到 Framework 里面的 Trace 点![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161742896.png)
    2. 使用插件后﻿，可以看到 Trace 中显示的信息多了很多（App 自身的代码逻辑，Framework 的代码没法插桩）![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161743198.png)
# 2. Android App 冷启动流程分析

本文以 **在桌面上冷启动一个 Android App 为例**，应用冷启动的整个流程包含了从用户触摸屏幕到应用完全显示的整个流程，其中涉及到

1. 触摸屏中断处理阶段
2. InputReader 和 InputDispatcher 处理 input 事件阶段
3. Launcher 处理 input 事件阶段
4. SystemServer 处理启动事件
5. 启动动画
6. 应用启动和自身逻辑阶段

上一篇文章有讲到响应速度问题，需要搞清楚 **起点** 和 **终点**，对于应用冷启动来说，**起点**就是 input 事件，**终点**就是应用完全展示给用户（用户可操作）

下面将从上面几个关键流程，通过 Systrace 的来介绍整个流程

## 2.1 触摸屏中断处理阶段[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-2/#2-1-%E8%A7%A6%E6%91%B8%E5%B1%8F%E4%B8%AD%E6%96%AD%E5%A4%84%E7%90%86%E9%98%B6%E6%AE%B5)

由于我们的案例是在桌面冷启动一个 App，那么在手指触摸手机屏幕的时候，触摸屏会触发中断，这个中断我们最早能在 Systrace 中看到的地方如下：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161743646.png)

对应的 cpu ss 区域和 中断区域（加了 irq 的 tag 才可以看到）

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161743523.png)

一般来说，点击屏幕会触发若干个中断，这些信号经过处理之后，触摸屏驱动会把这些点更新到 EventHub 中，让 InputReader 和 InputDIspatcher 进行进一步的处理。这一步一般不会出现什么问题，厂商这边对触摸屏的调教可能会关注这里

## 2.2 InputReader 和 InputDispatcher 处理 Input 事件阶段[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-2/#2-2-InputReader-%E5%92%8C-InputDispatcher-%E5%A4%84%E7%90%86-Input-%E4%BA%8B%E4%BB%B6%E9%98%B6%E6%AE%B5)

InputReader 和 InputDispatcher 这两个线程跑在 SystemServer 里面，专门负责处理 Input 事件，具体的流程可以参考[Android Systrace 基础知识 - Input 解读](https://www.androidperformance.com/2019/11/04/Android-Systrace-Input/) 这篇文章

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161744719.png)

这里由于我们是点击桌面上的一个 App 的图标，可以看到底层上报上来的事件包括一个 Input_Down 事件 + 若干个 Input Move 事件 + 一个 Input Up 事件，组成了一个完整的点击事件

由于 Launcher 在进程创建的时候就注册了 Input 监听，且此时 Launcher 在前台且可见，所以 Launcher 进程可以收到这些 Input 事件，并根据 Input 事件的类型进行处理，input 事件在 SystemServer 和 App 的流转在 Systrace 中的具体表现可以参考 [Android Systrace 基础知识 - Input 解读](https://www.androidperformance.com/2019/11/04/Android-Systrace-Input/) ，这里把核心的两张图放上来

### 2.2.1 Input 事件在 SystemServer 中流转[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-2/#2-2-1-Input-%E4%BA%8B%E4%BB%B6%E5%9C%A8-SystemServer-%E4%B8%AD%E6%B5%81%E8%BD%AC)

看下图即可，如果要看更详细的，可以查看 [Android Systrace 基础知识 - Input 解读](https://www.androidperformance.com/2019/11/04/Android-Systrace-Input/)

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161744104.png)


### 2.2.2 Input 事件在 Launcher 进程流转[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-2/#2-2-2-Input-%E4%BA%8B%E4%BB%B6%E5%9C%A8-Launcher-%E8%BF%9B%E7%A8%8B%E6%B5%81%E8%BD%AC)

看下图即可，如果要看更详细的，可以查看 [Android Systrace 基础知识 - Input 解读](https://www.androidperformance.com/2019/11/04/Android-Systrace-Input/)

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161744030.png)

## 2.3 Launcher 进程处理 Input 事件阶段[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-2/#2-3-Launcher-%E8%BF%9B%E7%A8%8B%E5%A4%84%E7%90%86-Input-%E4%BA%8B%E4%BB%B6%E9%98%B6%E6%AE%B5)

Launcher 处理 Input 事件也是响应时间的一个重要阶段，主要包括两个响应速度指标

1. 点击桌面到桌面第一帧响应（一般 Launcher 会在接收到 Down 事件的时候，将 App 图标置灰，以表示接收到了事件；有的定制桌面 App 图标会有一个缩小的动画，表示被按压）
2. 桌面第一帧响应到启动 App（这段时间指的是桌面在收到 Down 对 App 图标做处理后，到收到 Up 事件判断需要启动 App 的时间）

另外提一下，**滑动桌面到桌面第一帧响应时间**（这个指的是滑动桌面的场景，左右滑动桌面的时候，用高速相机拍摄，从手指动开始，到桌面动的第一帧的时间）也是一个很重要的**响应速度指标**，部分厂商也会在这方面做优化，感兴趣的可以自己试试主流厂商的桌面滑动场景（跟原生的机器对比 Systrace 即可）

在冷启动的场景里面，Launcher 在收到 up 事件后，会进行逻辑判断，然后启动对应的 App（这里主要是交给 AMS 来处理，又回到了 SystemServer 进程）

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161745508.png)

这个阶段通常也是做系统优化的会比较关注，做 App 的同学还不需要关注到这里（Launcher App 的除外）；另外在最新的版本，应用启动的动画是由 Launcher 和 SystemServer 共同完成的，目的就是可以做一些复杂的动画而没有割裂感，大家可以用慢镜头拍一下启动时候和退出应用的动画，可以看到有的应用图标是分层的，甚至会动，这是之前纯粹由 SystemServer 这边来做动画所办不到的

## 2.4 SystemServer 处理 StartActivity 阶段[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-2/#2-4-SystemServer-%E5%A4%84%E7%90%86-StartActivity-%E9%98%B6%E6%AE%B5)

SystemServer 处理主要是有2部分

1. 处理启动命令
2. 通知 Launcher 进入 Pause 状态
3. fork 新的进程

### 处理启动命令[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-2/#%E5%A4%84%E7%90%86%E5%90%AF%E5%8A%A8%E5%91%BD%E4%BB%A4)

这个 SystemServer 进程中的 Binder 调用就是 Launcher 通过 ActivityTaskManager.getService().startActivity 调用过来的

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161745655.png)

fork 新的进程，则是在判断启动的 Activity 的 App 进程没有启动后，需要首先启动进程，然后再启动 Activity，这里是冷启动和其他启动不一样的地方。fork 主要是 fork Zygote64 这个进程（部分 App 是 fork 的 Zygote32 ）

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161745999.png)

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161745288.png)

### Zygote 64 位进程执行 Fork 操作

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161745676.png)


![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161746494.png)

### 对应的 App 进程出现

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161746054.png)

对应的代码如下，这里就正式进入了 App 自己的进程逻辑了

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161746687.png)

应用启动后，SystemServer 会记录从 startActivity 被调用到应用第一帧显示的时长，在 Systrace 中的显示如下（**注意结尾是应用第一帧，如果应用启动的时候是 SplashActivity -> MainActivity，那么这里的结尾只是 SplashActivity，MainActivity 的完全启动需要自己查看**）

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161746822.png)

## 2.5 应用进程启动阶段[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-2/#2-5-%E5%BA%94%E7%94%A8%E8%BF%9B%E7%A8%8B%E5%90%AF%E5%8A%A8%E9%98%B6%E6%AE%B5)

通常的大型应用，App 冷启动通常包括下面三个部分，每一个部分耗时都会导致应用的整体启动速度变慢，所以在优化启动速度的时候，需要明确知道应用启动结束的点（需要跟测试沟通清楚，一般是界面保持稳定的那个点）

1. 应用进程启动到 SplashActivity 第一帧显示（部分 App 没有 SplashActivity，所以可以省略这一步，直接到进程启动到 主 Activit 第一帧显示 ）
2. SplashActivity 第一帧显示到主 Activity 第一帧显示
3. 主 Activity 第一帧显示到界面完全显示

下面针对这三个阶段来具体分析（当然你的 App 如果简单的话，可能没有 SplashActivity ，直接进的就是主 Activity，那么忽略第二步就可以了）

### 应用进程启动到 SplashActivity 第一帧显示[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-2/#%E5%BA%94%E7%94%A8%E8%BF%9B%E7%A8%8B%E5%90%AF%E5%8A%A8%E5%88%B0-SplashActivity-%E7%AC%AC%E4%B8%80%E5%B8%A7%E6%98%BE%E7%A4%BA)

由于是冷启动，所以 App 进程在 Fork 之后，需要首先执行 bindApplication ，这个也是区分冷热启动的一个重要的点。Application 的环境创建好之后，就开始组件的启动（这里是 Activity 组件，通过 Service、Broadcast、ContentProvider 组件启动的进程则会在 bindApplication 之后先启动这些组件）

Activity 的生命周期函数会在 Activity 组件创建的时候执行，包括 onStart、onCreate、onResume 等，然后还要经过一次 Choreographer#doFrame 的执行（包括 measure、layout、draw）以及 RenderThread 的初始化和第一帧任务的绘制，再加上 SurfaceFlinger 一个 Vsync 周期的合成，应用第一帧才会真正显示（也就是下图中 finishDrawing 的位置），这部分详细的流程可以查看 [Android Systrace 基础知识 - MainThread 和 RenderThread 解读](https://www.androidperformance.com/2019/11/06/Android-Systrace-MainThread-And-RenderThread/)

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161747381.png)

### SplashActivity 第一帧显示到主 Activity 第一帧显示[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-2/#SplashActivity-%E7%AC%AC%E4%B8%80%E5%B8%A7%E6%98%BE%E7%A4%BA%E5%88%B0%E4%B8%BB-Activity-%E7%AC%AC%E4%B8%80%E5%B8%A7%E6%98%BE%E7%A4%BA)

大部分的 App 都有 SplashActivity 来播放广告，播放完成之后才是真正的主 Activity 的启动，同样包括 Activity 组件的创建，包括 onStart、onCreate、onResume 、自有启动逻辑的执行、WebView 的初始化等等等等，直到主 Activity 的第一帧显示

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161747630.png)


### 主 Activity 第一帧显示到界面完全加载并显示[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-2/#%E4%B8%BB-Activity-%E7%AC%AC%E4%B8%80%E5%B8%A7%E6%98%BE%E7%A4%BA%E5%88%B0%E7%95%8C%E9%9D%A2%E5%AE%8C%E5%85%A8%E5%8A%A0%E8%BD%BD%E5%B9%B6%E6%98%BE%E7%A4%BA)

一般来说，主 Activity 需要多帧才能显示完全，因为有很多资源（最常见的是图片）是异步加载的，第一帧可能只加载了一个显示框架、而其中的内容在准备好之后才会显示出来。这里也可以看到，通过 Systrace 不是很方便来判断应用冷启动的终点（除非你跟测试约定好，在某个 View 显示之后就算启动完成，然后你在这个 View 里面打个 Systrace 的 Tag，通过跟踪这个 Tag 就可以粗略判断具体 Systrace 里面哪一帧是启动完成的点）

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161747717.png)


我制作了一个 Systrace + 截图的方式，来进行演示，方便你了解 App 启动各个阶段都对应在 Systrace 的哪里（使用的是一个开源的 WanAndroid 客户端）

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161748583.png)


# End

本文重点放在了如何在 Systrace 中展示 App 的完整冷启动流程，方便大家在做 App 的启动优化的时候，可以通过 Systrace 来快速定位到启动瓶颈，也方便进行竞品的对比和分析，

1. 至于如何分析，可以查看 [Systrace 响应速度实战 1 ：了解响应速度原理](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-1/) 的分析套路部分
2. 至于如何优化，可以查看 [Android App 启动优化全记录](https://www.androidperformance.com/2019/11/18/Android-App-Lunch-Optimize/) 这篇文章，这里就不再重复了。不过随着技术的发展，有些优化手段会消失，而会有新的优化手段冒出来，我也会对这篇文章进行维护，如果大家发现新的优化技术，麻烦博客留言或者加微信（553000664）通知我，我来进行调研和更新

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

1. [Systrace 响应速度实战 2 ：响应速度实战分析-以启动速度为例](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-2/)


