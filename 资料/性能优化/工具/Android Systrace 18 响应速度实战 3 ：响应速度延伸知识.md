---
author: zjmantou
title: Android Systrace 18 响应速度实战 3 ：响应速度延伸知识
time: 2024-03-16 周六
tags:
  - Android
  - 资料
  - 性能优化
  - Systrace
  - Tools
---
在讨论 Android 性能问题的时候，卡顿、响应速度、ANR 这三个性能相关的知识点通常会放到一起来讲，因为引起卡顿、响应慢、ANR 的原因类似，只不过根据重要程度，被人为分成了卡顿、响应慢、ANR 三种，所以我们可以定义广义上的卡顿，包含了卡顿、响应慢和 ANR 三种，所以如果用户反馈说手机卡顿或者 App 卡顿，大部分情况下都是广义上的卡顿，需要搞清楚，到底出现了哪一种问题

如果是动画播放卡顿、列表滑动卡顿这种，我们一般定义为 狭义的卡顿，对应的英文描述我觉得应该是 Jank；如果是应用启动慢、亮灭屏慢、场景切换慢，我们一般定义为 响应慢，对应的英文描述我觉得应该是 Slow ；如果是发生了 ANR，那就是 **应用无响应问题** 。三种情况所对应的分析方法和解决方法不太一样，所以需要分开来讲

另外在 App 或者厂商内部，卡顿、响应速度、ANR 这几个性能指标都是有单独的标准的，比如 掉帧率、启动速度、ANR 率等，所以针对这些性能问题的分析和优化能力，对开发者来说就非常重要了

本文是响应速度系列的第三篇，主要是讲在使用 Systrace 分析应用响应速度问题的时候，其中的一些延伸知识，包括启动速度测试、Log 输出解读、Systrace 状态解读、三方启动库等内容

Systrace (Perfetto) 工具的基本使用如果还不是很熟悉，那么需要优先去补一下 [Systrace 基础知识系列](https://www.androidperformance.com/2019/05/26/Android_Systrace_0/)，本文假设你已经熟悉 Systrace(Perfetto)的使用了

# 1. Systrace 中进程三种状态解读

Systrace 中，进程的任务最常见的有三种状态：Sleep、Running、Runnable。在优化的过程中，这几个状态也需要我们关注。进程任务状态在最上面，以颜色来做区分：

1. 绿色：Running
2. 蓝色：Runnable
3. 白色：Sleep

## 1.1 如何分析 Sleep 状态的 Task[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-3/#1-1-%E5%A6%82%E4%BD%95%E5%88%86%E6%9E%90-Sleep-%E7%8A%B6%E6%80%81%E7%9A%84-Task)

一般白色的 Sleep 有两种，即应用主动 Sleep 和被动 Sleep

1. nativePoll 这种，一般属于主动 Sleep，因为没有消息处理了，所以进入 Sleep 状态等待 Message，这种一般是正常的，我们不需要去关注。比如两帧之间的那段，就是主动 sleep 的
2. 被动 Sleep 一般是由用户主动调用 sleep，或者用 Binder 与其他进程进行通信，这个是我们最常见的，也是分析性能问题的时候经常会遇到的，需要重点关注

如下图，这种在启动过程中，有较长时间的 sleep 情况，一般下面就可以看到是否在进行 Binder 通信，如果在启动过程中有频繁的 Binder 通信，那么应用等待的时间就会变长，导致响应时间变慢

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161750811.png)

这种一般可以点击这个 Task 最下面的 binder transaction 来查看 Binder 调用信息，比如

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161750226.png)


有时候没有 Binder 信息，是被其他的等待的线程唤醒，那么可以查看唤醒信息，也可以找到应用是在等待什么

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161750123.png)


放大上图中我们点击的 Runnable 的地方

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161751735.png)

## 1.2 如何分析 Running 状态的 Task[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-3/#1-2-%E5%A6%82%E4%BD%95%E5%88%86%E6%9E%90-Running-%E7%8A%B6%E6%80%81%E7%9A%84-Task)

Running 状态的任务就是目前在 CPU 某一个核心上运行的任务，如果某一段任务是 Running 状态，且耗时变长，那么需要分析：

1. 是否应用的本身逻辑耗时，比如新增了某些代码逻辑
2. 是否跑在了对应的核心上

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161751136.png)
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161751504.png)


在某些 Android 机器上，大家一般会对 App 的主线程和渲染线程进行调度方面的优化：一般前台应用的 UI Thread 和 RenderThread 都是跑在大核上的

## 1.3 如何分析 Runnable 状态的 Task[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-3/#1-3-%E5%A6%82%E4%BD%95%E5%88%86%E6%9E%90-Runnable-%E7%8A%B6%E6%80%81%E7%9A%84-Task)

一个 Task 要从 Sleep 状态转到 Running 状态，必须先变成 Runnable 状态，其状态转换图如下

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161753510.png)


在 Systrace 上的表现如下

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161753377.png)


正常情况下，应用进入 Runnable 状态之后，会马上被调度器调度，进入 Running 状态，开始干活；但是在系统繁忙的时候，应用就会有大量的时间在 Runnable 状态，因为 cpu 已经跑满，各种任务都需要排队等待调度

如果应用启动的时候出现大量的 Runnable 任务，那么需要查看系统的状态

# 2. TraceView 工具在响应速度方面的使用

TraceView 指的是我们在 AS Profiler 里面抓取 CPU 信息的时候出现的那个，大家看下面的截图就知道了

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161754421.png)


## 2.1 如何抓取应用启动时候的 TraceView[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-3/#2-1-%E5%A6%82%E4%BD%95%E6%8A%93%E5%8F%96%E5%BA%94%E7%94%A8%E5%90%AF%E5%8A%A8%E6%97%B6%E5%80%99%E7%9A%84-TraceView)

使用下面的命令可以抓取应用的冷启动，这些命令也可以分开执行，需要把里面的包名和 Activity 名切换成自己应用的包名

```shell
adb shell am start -n com.aboback.wanandroidjetpack/.splash.SplashActivity --start-profiler /data/local/tmp/traceview.trace --sampling 1 && sleep 10 && adb shell am profile stop com.aboback.wanandroidjetpack && adb pull /data/local/tmp/traceview.trace .

```

或者分开执行上面的命令

```shell
// 1. 冷启动 App，sampleing = 1 意思是 1ms 采样一次  
adb shell am start -n com.aboback.wanandroidjetpack/.splash.SplashActivity --start-profiler /data/local/tmp/traceview.trace --sampling 1  
  
// 2. 等待应用完全启动之后，结束 profile  
adb shell am profile stop com.aboback.wanandroidjetpack  
  
// 3. 将 Trace 文件从手机里面 pull 出来  
adb pull /data/local/tmp/traceview.trace .  
  
// 4. 使用 Android Studio 打开 traceview.trace 文件
```

## 2.2 TraceView 工具怎么看[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-3/#2-2-TraceView-%E5%B7%A5%E5%85%B7%E6%80%8E%E4%B9%88%E7%9C%8B)

抓出来的 TraceView 可以直接在 Android Studio 中打开

其中图里面用绿色标记的函数，就是应用自己的函数，黄色标注的是系统的函数

Application.onCreate

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161755309.png)


Activity.onCreate

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161755846.png)

doFrame

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161756206.png)


WebView 初始化

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161756875.png)

## 2.3 TraceView 工具的弊端[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-3/#2-3-TraceView-%E5%B7%A5%E5%85%B7%E7%9A%84%E5%BC%8A%E7%AB%AF)

由于采样比较细，所以会性能损耗比较大，所以抓出来的 TraceView，其中每个方法的执行时间是不准的，所以不可用作为真实的时间参考，但是可以用来定位具体的函数调用栈。

需要跟 Systrace 来进行互补

# 3. SimplePerf 工具在启动速度分析的使用

使用 SimplePerf 工具也可以抓取启动时候的堆栈信息，既包括 Java 也包括 Native

比如我们要抓取 com.aboback.wanandroidjetpack 这个应用的冷启动，可以执行下面的命令（SimplePerf 的环境初始化参考 [https://android.googlesource.com/platform/system/extras/+/master/simpleperf/doc/android_application_profiling.md](https://android.googlesource.com/platform/system/extras/+/master/simpleperf/doc/android_application_profiling.md) 这篇文章 ，其中 app_profiler.py 就是 SimplePerf 的工具）

```python
python app_profiler.py -p com.aboback.wanandroidjetpack
```

执行上面的命令之后，需要手动在手机上启动 App，然后主动结束

```shell
$ python app_profiler.py -p com.aboback.wanandroidjetpack  
INFO:root:prepare profiling  
INFO:root:start profiling1  
INFO:root:run adb cmd: ['adb', 'shell', '/data/local/tmp/simpleperf', 'record', '-o', '/data/local/tmp/perf.data', '-e task-clock:u -f 1000 -g --duration 10', '--log', 'info', '--app', 'com.aboback.wanandroidjetpack'] simpleperf I environment.cpp:601] Waiting for process of app com.aboback.wanandroidjetpack  
simpleperf I environment.cpp:593] Got process 32112 for package com.aboback.wanandroidjetpack
```

抓取结束之后，调用解析脚本来生成 html 报告

```python
python report_html.py
```

就会得到下面这个

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161757969.png)


不仅可以看到 Java 层的堆栈，也可以看到 Native 的堆栈，这里只是简单的使用，更详细的方法可以参考下面几个文档

SimplePerf 初步试探 [https://android.googlesource.com/platform/system/extras/+/master/simpleperf/doc/README.md](https://android.googlesource.com/platform/system/extras/+/master/simpleperf/doc/README.md)

1. Android application profiling [https://android.googlesource.com/platform/system/extras/+/master/simpleperf/doc/android_application_profiling.md](https://android.googlesource.com/platform/system/extras/+/master/simpleperf/doc/android_application_profiling.md)
2. Android platform profiling [https://android.googlesource.com/platform/system/extras/+/master/simpleperf/doc/android_platform_profiling.md](https://android.googlesource.com/platform/system/extras/+/master/simpleperf/doc/android_platform_profiling.md)
3. Executable commands reference [https://android.googlesource.com/platform/system/extras/+/master/simpleperf/doc/executable_commands_reference.md](https://android.googlesource.com/platform/system/extras/+/master/simpleperf/doc/executable_commands_reference.md)
4. Scripts reference [https://android.googlesource.com/platform/system/extras/+/master/simpleperf/doc/scripts_reference.md](https://android.googlesource.com/platform/system/extras/+/master/simpleperf/doc/scripts_reference.md)

# 4. 其他组件启动时在 Systrace 中的位置

## 4.1 Service 的启动[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-3/#4-1-Service-%E7%9A%84%E5%90%AF%E5%8A%A8)

```java
public final void scheduleCreateService(IBinder token,  
        ServiceInfo info, CompatibilityInfo compatInfo, int processState) {  
    updateProcessState(processState, false);  
    CreateServiceData s = new CreateServiceData();  
    s.token = token;  
    s.info = info;  
    s.compatInfo = compatInfo;  
    sendMessage(H.CREATE_SERVICE, s);  
}  
  
public final void scheduleBindService(IBinder token, Intent intent,  
        boolean rebind, int processState) {  
    updateProcessState(processState, false);  
    BindServiceData s = new BindServiceData();  
    s.token = token;  
    s.intent = intent;  
    s.rebind = rebind;  
    sendMessage(H.BIND_SERVICE, s);  
}
```

可以看到，代码执行都是往 H 这个 Handler 中发送 Message,所以如果我们在代码里面启动 Service，并不是马上就执行的，而是由 MessageQueue 里面的 Message 顺序决定的

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161759228.png)


放大真正执行的部分可以看到，其执行的时机是在 MessageQueue 按照 Message 的顺序执行(这里是在应用第一帧执行结束后)，后面的 Message 就是应用自己的 Message、启动 Service、执行广播接收器

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161759075.png)


## 4.2 执行自己的 Message[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-3/#4-2-%E6%89%A7%E8%A1%8C%E8%87%AA%E5%B7%B1%E7%9A%84-Message)

执行自定义的 Message 在 Systrace 中的显示

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161759304.png)

## 4.3 启动 Service[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-3/#4-3-%E5%90%AF%E5%8A%A8-Service)

Service 启动在 Systrace 中的显示

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161759873.png)


## 4.4 启动 BroadcastReceiver[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-3/#4-4-%E5%90%AF%E5%8A%A8-BroadcastReceiver)

执行 Receiver 在 Systrace 中的显示

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161759678.png)


Broadcast 的注册：一般是在 Activity 生命周期函数中注册，在哪里注册就在哪里执行

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161800465.png)

## 4.5 ContentProvider 的启动时机[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-3/#4-5-ContentProvider-%E7%9A%84%E5%90%AF%E5%8A%A8%E6%97%B6%E6%9C%BA)

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161800963.png)

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161800469.png)

# 5. AppStartup 是否能优化启动速度？

#### 三方库的初始化[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-3/#%E4%B8%89%E6%96%B9%E5%BA%93%E7%9A%84%E5%88%9D%E5%A7%8B%E5%8C%96)

很多三方库都需要在 Application 中进行初始化，并顺便获取到 Application 的上下文

但是也有的库不需要我们自己去初始化，它偷偷摸摸就给初始化了，用到的方法就是使用 ContentProvider 进行初始化，定义一个 ContentProvider，然后在 onCreate 拿到上下文，就可以进行三方库自己的初始化工作了。而在 APP 的启动流程中，有一步就是要执行到程序中所有注册过的 ContentProvider 的 onCreate 方法，所以这个库的初始化就默默完成了。

这种做法确实给集成库的开发者们带来了很大的便利，现在很多库都用到了这种方法，比如 Facebook，Firebase，WorkManager

ContentProvider 的初始化时机如下：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161801515.png)


但是当大部分三方库使用这种方法初始化的时候，就会有下面几个问题

1. 启动过程中的 ContentProvider 过多
2. 应用开发者无法控制使用这种方式初始化的库的初始化时机
3. 无法处理这些三方库的依赖

#### AppStartup 库[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-3/#AppStartup-%E5%BA%93)

针对上面的情况，Google 推出了 [AppStartup](https://developer.android.google.cn/topic/libraries/app-startup) 库，AppStartup 库的优点

- 可以共享单个 Contentprovider
- 可以明确地设置初始化顺序
- 通过这个库可以移除三方库的 ContentProvider 启动时候自动初始化的步骤，手动通过 LazyLoad 的方式启动，这样可以起到优化启动速度的作用

根据测算结果来看，使用 AppStartup 库并不能显著加快应用启动速度，除非你有非常多 (50+）的 ContentProvider 在应用启动的时候初始，那么 AppStartup 才会有比较明显的效果

如果三方的 SDK 使用 ContentProvider 初始化耗时，那么可以考虑针对这个 ContentProvider 进行延迟初始化，比如

```xml
<provider  
    android:name="androidx.startup.InitializationProvider"  
    android:authorities="${applicationId}.androidx-startup"  
    android:exported="false"  
    tools:node="merge">  
    <meta-data android:name="com.example.ExampleLoggerInitializer"  
              tools:node="remove" />  
</provider>
```

ExampleLoggerInitializer 的 meta-data 当中加入了一个 tools:node=”remove”的标记

### 总结[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-3/#%E6%80%BB%E7%BB%93)

1. App Startup 的设计是为了解决一个问题：即不同的库使用不同的 ContentProvider 进行初始化，导致 ContentProvider 太多，管理杂乱，影响耗时的问题
2. App Startup 具体能减少多少耗时时间:根据测试，如果二三十个三方库都集成了 App Startup，减少的耗时大概在 20ms 以内
3. App Startup 的使用场景应该
    1. APK 有很多的 ContentProvider 在启动时候初始化
    2. APK 中有的三方库 ContentProvider 初始化很耗时，但是又不是必须要在启动的时候初始，可以按需初始化
    3. 应用开发者想自己控制各个库的初始化时机或者初始化顺序

#### 需要 App 开发同学验证[](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-3/#%E9%9C%80%E8%A6%81-App-%E5%BC%80%E5%8F%91%E5%90%8C%E5%AD%A6%E9%AA%8C%E8%AF%81)

1. 检查打包出来的 apk 的配置文件里面看一下，有多少个三方库是利用 ContentProvider 初始化的（或者在 AS 的 src\main\AndroidManifest.xml 文件最下面打开 Merged Manifest 标签查看）
2. 确认这些 ContentProvider 在启动时候的耗时
3. 确认哪些 ContentProvider 可以延迟加载或者用时加载
4. 如果需要的话，接入 AppStartup 库

# 6. IdleHandler 在 App 启动场景下的使用

在启动优化的过程中，idleHandler 可以在 MessageQueue 空闲的时候执行任务，如下图，可以很清晰地查看 idleHandler 的执行时机

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161803676.png)

其使用场景如下：

1. **在启动的过程中，可以借助 idleHandler 来做一些延迟加载的事情。** 比如在启动过程中 Activity 的 onCreate 里面 addIdleHandler，这样在 Message 空闲的时候，可以执行这个任务![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161803351.png)

**进行启动时间统计**：比如在页面完全加载之后，调用 activity.reportFullyDrawn 来告知系统这个 Activity 已经完全加载，用户可以使用了，比如下面的例子，在主页的 List 加载完成后，调用 activity.reportFullyDrawn![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161804428.png)

其对应的 Systrace 如下

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161804199.png)


这时候得到的应用的冷启动时间才是正常的

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161804053.png)

另外系统有些功能，也会依赖于 FullyDrawn，所以建议主动上报（即主动在 App 完全启动后调用 activity.reportFullyDrawn）

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

1. [Systrace 响应速度实战 3 ：响应速度延伸知识](https://www.androidperformance.com/2021/09/13/android-systrace-Responsiveness-in-action-3/)
