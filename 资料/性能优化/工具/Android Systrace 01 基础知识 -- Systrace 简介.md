---
author: zjmantou
title: Android Systrace 01 基础知识 -- Systrace 简介
time: 2024-03-15 周五
tags:
  - Android
  - 资料
  - 性能优化
  - Systrace
  - Tools
---


# 正文

Systrace 是 Android4.1 中新增的性能数据采样和分析工具。它可帮助开发者收集 Android 关键子系统（如 SurfaceFlinger/SystemServer/Kernel/Input/Display 等 Framework 部分关键模块、服务，View系统等）的运行信息，从而帮助开发者更直观的分析系统瓶颈，改进性能。

Systrace 的功能包括跟踪系统的 I/O 操作、内核工作队列、CPU 负载以及 Android 各个子系统的运行状况等。在 Android 平台中，它主要由3部分组成：

- **内核部分**：Systrace 利用了 Linux Kernel 中的 ftrace 功能。所以，如果要使用 Systrace 的话，必须开启 kernel 中和 ftrace 相关的模块。
- **数据采集部分**：Android 定义了一个 Trace 类。应用程序可利用该类把统计信息输出给ftrace。同时，Android 还有一个 atrace 程序，它可以从 ftrace 中读取统计信息然后交给数据分析工具来处理。
- **数据分析工具**：Android 提供一个 systrace.py（ python 脚本文件，位于 Android SDK目录/platform-tools/systrace 中，其内部将调用 atrace 程序）用来配置数据采集的方式（如采集数据的标签、输出文件名等）和收集 ftrace 统计数据并生成一个结果网页文件供用户查看。 从本质上说，Systrace 是对 Linux Kernel中 ftrace 的封装。应用进程需要利用 Android 提供的 Trace 类来使用 Systrace.  
    关于 Systrace 的官方介绍和使用可以看这里：[Systrace](http://developer.android.com/tools/help/systrace.html "SysTrace官方介绍")

> Note : 最新版本的 platform-tools 里面已经移除了 Systrace 工具，Google 推荐使用 Perfetto 来抓 Trace
> 
> 个人建议：Systrace 和 Perfetto 都可以用，哪个用着顺手就用哪个，不过最终 Google 是要用 Perfetto 来替代 Systrace 的，所以可以把默认的 Trace 打开工具切换成 Perfetto UI
> 
> Perfetto 相比 Systrace 最大的改进是可以支持长时间数据抓取，这是得益于它有一个可在后台运行的服务，通过它实现了对收集上来的数据进行 Protobuf 的编码并存盘。从数据来源来看，核心原理与 Systrace 是一致的，也都是基于 Linux 内核的 Ftrace 机制实现了用户空间与内核空间关键事件的记录（ATRACE、CPU 调度）。Systrace 提供的功能 Perfetto 都支持，由此才说 Systrace 最终会被 Perfetto 替代。
> 
> Perfetto 所支持的数据类型、获取方法，以及分析方式上看也是前所未有的全面，它几乎支持所有的类型与方法。数据类型上通过 ATRACE 实现了 Trace 类型支持，通过可定制的节点读取机制实现了 Metric 类型的支持，在 UserDebug 版本上通过获取 Logd 数据实现了 Log 类型的支持。
> 
> 你可以通过 Perfetto.dev 网页、命令行工具手动触发抓取与结束，通过设置中的开发者选项触发长时间抓取，甚至你可以通过框架中提供的 Perfetto Trigger API 来动态开启数据抓取，基本上涵盖了我们在项目上能遇到的所有的情境。
> 
> 在数据分析层面，Perfetto 提供了类似 Systrace 操作的数据可视化分析网页，但底层实现机制完全不同，最大的好处是可以支持超大文件的渲染，这是 Systrace 做不到的（超过 300M 以上时可能会崩溃、可能会超卡）。在这个可视化网页上，可以看到各种二次处理的数据、可以执行 SQL 查询命令、甚至还可以看到 logcat 的内容。Perfetto Trace 文件可以转换成基于 SQLite 的数据库文件，既可以现场敲 SQL 也可以把已经写好的 SQL 形成执行文件。甚至你可以把他导入到 Jupyter 等数据科学工具栈，将你的分析思路分享给其他伙伴。
> 
> 比如你想要计算 SurfaceFlinger 线程消耗 CPU 的总量，或者运行在大核中的线程都有哪一些等等，可以与领域专家合作，把他们的经验转成 SQL 指令。如果这个还不满足你的需求， Perfetto 也提供了 Python API，将数据导出成 DataFrame 格式近乎可以实现任意你想要的数据分析效果。
> 
> 可以在 Android 性能优化的术、道、器 这篇文章中查看各个工具的介绍和使用，以及他们的优劣比。
> 
> 笔者后续会持续更新 Perfetto 系统，后续所有的图例也都会用 Perfetto 来演示。

### **Systrace 的设计思路**[](https://www.androidperformance.com/2019/05/28/Android-Systrace-About/#Systrace-%E7%9A%84%E8%AE%BE%E8%AE%A1%E6%80%9D%E8%B7%AF)

在**系统的一些关键操作**（比如 Touch 操作、Power 按钮、滑动操作等）、**系统机制**（input 分发、View 绘制、进程间通信、进程管理机制等）、**软硬件信息**（CPU 频率信息、CPU 调度信息、磁盘信息、内存信息等）的关键流程上，插入类似 Log 的信息，我们称之为 TracePoint（本质是 Ftrace 信息），通过这些 TracePoint 来展示一个核心操作过程的执行时间、某些变量的值等信息。然后 Android 系统把这些散布在各个进程中的 TracePoint 收集起来，写入到一个文件中。导出这个文件后，Systrace 通过解析这些 TracePoint 的信息，得到一段时间内整个系统的运行信息。

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403152219462.png)

Android 系统中，一些重要的模块都已经默认插入了一些 TracePoint，通过 TraceTag 来分类，其中信息来源如下

1. Framework Java 层的 TracePoint 通过 android.os.Trace 类完成
2. Framework Native 层的 TracePoint 通过 ATrace 宏完成
3. App 开发者可以通过 android.os.Trace 类自定义 Trace

这样 Systrace 就可以把 Android 上下层的所有信息都收集起来并集中展示，对于 Android 开发者来说，Systrace 最大的作用就是把整个 Android 系统的运行状态，从黑盒变成了白盒。全局性和可视化使得 Systrace 成为 Android 开发者在分析复杂的性能问题的时候的首选。

### 实践中的应用情况[](https://www.androidperformance.com/2019/05/28/Android-Systrace-About/#%E5%AE%9E%E8%B7%B5%E4%B8%AD%E7%9A%84%E5%BA%94%E7%94%A8%E6%83%85%E5%86%B5)

解析后的 Systrace 由于有大量的系统信息，天然适合分析 Android App 和 Android 系统的性能问题， Android 的 App 开发者、系统开发者、Kernel 开发者都可以使用 Systrace 来分析性能问题。

1. 从技术角度来说，Systrace 可覆盖性能涉及到的 **响应速度** 、**卡顿丢帧**、 **ANR** 这几个大类。
2. 从用户角度来说，Systrace 可以分析用户遇到的性能问题，包括但不限于:
    1. 应用启动速度问题，包括冷启动、热启动、温启动
    2. 界面跳转速度慢、跳转动画卡顿
    3. 其他非跳转的点击操作慢（开关、弹窗、长按、选择等）
    4. 亮灭屏速度慢、开关机慢、解锁慢、人脸识别慢等
    5. 列表滑动卡顿
    6. 窗口动画卡顿
    7. 界面加载卡顿
    8. 整机卡顿
    9. App 点击无响应、卡死闪退

在遇到上述问题后，可以使用多种方式抓取 Systrace ，将解析后的文件在 Chrome 打开，然后就可以进行分析

## Systrace 简单使用[](https://www.androidperformance.com/2019/05/28/Android-Systrace-About/#Systrace-%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8)

使用 Systrace 前，要先了解一下 Systrace 在各个平台上的使用方法，鉴于大家使用Eclipse 和 Android Studio 的居多，所以直接摘抄官网关于这个的使用方法，不过不管是什么工具，流程是一样的：

- 手机准备好你要进行抓取的界面
- 点击开始抓取(命令行的话就是开始执行命令)
- 手机上开始操作(不要太长时间)
- 设定好的时间到了之后，会将生成 Trace.html 文件，使用 **Chrome** 将这个文件打开进行分析

一般抓到的 Systrace 文件如下

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403152219385.png)

## 使用命令行工具抓取 Systrace[](https://www.androidperformance.com/2019/05/28/Android-Systrace-About/#%E4%BD%BF%E7%94%A8%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%B7%A5%E5%85%B7%E6%8A%93%E5%8F%96-Systrace)

命令行形式比较灵活，速度也比较快，一次性配置好之后，以后再使用的时候就会很快就出结果（**强烈推荐**）  
Systrace 工具在 Android-SDK 目录下的 platform-tools 里面（**最新版本的 platform-tools 里面已经移除了 systrace 工具，需要下载老版本的 platform-tools ，33 之前的版本**，可以在这里下载：[https://androidsdkmanager.azurewebsites.net/Platformtools](https://androidsdkmanager.azurewebsites.net/Platformtools) ）,下面是简单的使用方法

```bash
$ cd android-sdk/platform-tools/systrace  
$ python systrace.py
```

可以在 Bash 中配置好对应的路径和 Alias，使用起来还是很快速的。另外 User 版本所抓的 Systrce 文件所包含的信息,是比 eng 版本或者 Userdebug 版本要少的,建议使用 Userdebug 版本的机器来进行 debug,这样既保证了性能,又能有比较详细的输出结果.

抓取结束后，会生成对应的 Trace.html 文件，注意这个文件只能被 Chrome 打开。关于如何分析 Trace 文件，我们下面的章节会讲。不论使用那种工具，在抓取之前都可以选择参数，下面说一下这些参数的意思：

```shell
 -a appname      enable app-level tracing for a comma separated list of cmdlines; * is a wildcard matching any process  
 -b N            use a trace buffer size of N KB  
 -c              trace into a circular buffer  
 -f filename     use the categories written in a file as space-separated  
                   values in a line  
 -k fname,...    trace the listed kernel functions  
 -n              ignore signals  
 -s N            sleep for N seconds before tracing [default 0]  
 -t N            trace for N seconds [default 5]  
 -z              compress the trace dump  
 --async_start   start circular trace and return immediately  
 --async_dump    dump the current contents of circular trace buffer  
 --async_stop    stop tracing and dump the current contents of circular  
                   trace buffer  
 --stream        stream trace to stdout as it enters the trace buffer  
                   Note: this can take significant CPU time, and is best  
                   used for measuring things that are not affected by  
                   CPU performance, like pagecache usage.  
 --list_categories  
                 list the available tracing categories  
-o filename      write the trace to the specified file instead  
                   of stdout.
```

上面的参数虽然比较多，但使用工具的时候不需考虑这么多，在对应的项目前打钩即可，命令行的时候才会去手动加参数，我们一般会把这个命令配置成 Alias，比如（下面列出的 am，binder_driver 这些，不同的手机、root 和非 root，会有一些不同，可以查看下一节，使用 adb shell atrace –list_categories 来查看你的手机支持的 tag）：

```shell
alias st-start='python /path-to-sdk/platform-tools/systrace/systrace.py'  
alias st-start-gfx-trace = ‘st-start -t 8 am,binder_driver,camera,dalvik,freq,gfx,hal,idle,input,memory,memreclaim,res,sched,sync,view,webview,wm,workq,binder’
```

这样在使用的时候，可以直接敲 **st-start** 即可，当然为了区分和保持各个文件，还需要加上 **-o xxx.html** .上面的命令和参数不必一次就理解，只需要记住如何简单使用即可，在分析的过程中，这些东西都会慢慢熟悉的。

一般来说比较常用的是

1. -o : 指示输出文件的路径和名字
2. -t : 抓取时间(最新版本可以不用指定, 按 Enter 即可结束)
3. -b : 指定 buffer 大小 (一般情况下,默认的 Buffer 是够用的,如果你要抓很长的 Trae , 那么建议调大 Buffer )
4. -a : 指定 app 包名 (如果要 Debug 自定义的 Trace 点, 记得要加这个)

# 查看支持的 TAG

Systrace 默认支持的 TAG，可以通过下面的命令来进行抓取，不同厂商的机器可能有不同的配置，在使用的时候可以根据自己的需求来进行选择和配置，TAG 选的少的话，Trace 文件的体积也会相应的变小，但是抓取的内容也会相应变少。Trace 文件大小会影响其在 Chrome 中打开后的操作性能，所以这个需要自己取舍

以我手上的 Android 12 的机器为例，可以看到这台机器支持下面的 tag（左边是 tag 名，右边是解释）

```shell
$ adb shell atrace --list_categories  
gfx - Graphics  
input - Input  
view - View System  
webview - WebView  
wm - Window Manager  
am - Activity Manager  
sm - Sync Manager  
audio - Audio  
video - Video  
camera - Camera  
hal - Hardware Modules  
res - Resource Loading  
dalvik - Dalvik VM  
rs - RenderScript  
bionic - Bionic C Library  
power - Power Management  
pm - Package Manager  
ss - System Server  
database - Database  
network - Network  
adb - ADB  
vibrator - Vibrator  
aidl - AIDL calls  
nnapi - NNAPI  
rro - Runtime Resource Overlay  
pdx - PDX services  
sched - CPU Scheduling  
irq - IRQ Events  
i2c - I2C Events  
freq - CPU Frequency  
idle - CPU Idle  
disk - Disk I/O  
sync - Synchronization  
workq - Kernel Workqueues  
memreclaim - Kernel Memory Reclaim  
regulators - Voltage and Current Regulators  
binder_driver - Binder Kernel driver  
binder_lock - Binder global lock trace  
pagecache - Page cache  
memory - Memory  
thermal - Thermal event  
freq - CPU Frequency and System Clock (HAL)  
gfx - Graphics (HAL)  
ion - ION Allocation (HAL)  
memory - Memory (HAL)  
sched - CPU Scheduling and Trustzone (HAL)  
thermal_tj - Tj power limits and frequency (HAL)
```

# 原文链接

1. [Systrace 简介](https://www.androidperformance.com/2019/05/28/Android-Systrace-About/)

