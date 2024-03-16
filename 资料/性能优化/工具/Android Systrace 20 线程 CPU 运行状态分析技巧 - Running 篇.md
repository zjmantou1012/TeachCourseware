---
author: zjmantou
title: Android Systrace 20 线程 CPU 运行状态分析技巧 - Running 篇
time: 2024-03-16 周六
tags:
  - Android
  - 资料
  - 性能优化
  - Systrace
  - Tools
---

本文是 Systrace 线程 CPU 运行状态分析技巧系列的第二篇，主要分析了 Systrace 中 cpu 的 Running 状态出现的原因和 Running 过长时的一些优化思路。

本系列的目的是通过 Systrace 这个工具，从另外一个角度来看待 Android 系统整体的运行，同时也从另外一个角度来对 Framework 进行学习。也许你看了很多讲 Framework 的文章，但是总是记不住代码，或者不清楚其运行的流程，也许从 Systrace 这个图形化的角度，你可以理解的更深入一些。Systrace 基础和实战系列大家可以在 [Systrace 基础知识 - Systrace 预备知识](https://www.androidperformance.com/2019/07/23/Android-Systrace-Pre/) 或者 [博客文章目录](https://www.androidperformance.com/2019/12/01/BlogMap/) 这里看到完整的目录

1. [Systrace 线程 CPU 运行状态分析技巧 - Runnable 篇](https://www.androidperformance.com/2022/01/21/android-systrace-cpu-state-runnable/)
2. [Systrace 线程 CPU 运行状态分析技巧 - Running 篇](https://www.androidperformance.com/2022/03/13/android-systrace-cpu-state-running/)
3. [Systrace 线程 CPU 运行状态分析技巧 - Sleep 和 Uninterruptible Sleep 篇](https://www.androidperformance.com/2022/03/13/android-systrace-cpu-state-sleep/)

# Running 时间长

## 显示方式[](https://www.androidperformance.com/2022/03/13/android-systrace-cpu-state-running/#%E6%98%BE%E7%A4%BA%E6%96%B9%E5%BC%8F)

Trace 中显示绿色，表示线程处于运行态

## 原因 1: 代码本身复杂度高，执行耗时久[](https://www.androidperformance.com/2022/03/13/android-systrace-cpu-state-running/#%E5%8E%9F%E5%9B%A0-1-%E4%BB%A3%E7%A0%81%E6%9C%AC%E8%BA%AB%E5%A4%8D%E6%9D%82%E5%BA%A6%E9%AB%98%EF%BC%8C%E6%89%A7%E8%A1%8C%E8%80%97%E6%97%B6%E4%B9%85)

这是最常见的一种方式，当然不排除平台有bug，有时候厂商在libc、syscal等高频核心函数，加了一些逻辑导致了代码运行时间长。

优化建议: 优化逻辑、算法，降低复杂度。为了进一步判断具体是哪个函数耗时，可使用 AS CPU Profiler、simpleperf，或者自己通过 Trace.begin/end() API 添加更多 tracepoint 点

当然不排除有的时候平台有bug，在关键的libc或内核函数加了一些逻辑

## 原因 2: 代码以解释方式执行[](https://www.androidperformance.com/2022/03/13/android-systrace-cpu-state-running/#%E5%8E%9F%E5%9B%A0-2-%E4%BB%A3%E7%A0%81%E4%BB%A5%E8%A7%A3%E9%87%8A%E6%96%B9%E5%BC%8F%E6%89%A7%E8%A1%8C)

Trace 中看到 「Compiling」字眼时可能意味着它是解释执行方式进行。刚安装的应用（未做 odex）的程序经常会出现这种情况

优化建议: 使用 dex2oat 之后的版本试试，解释执行方式下的低性能暂无改善方法，除非执行 dex2oat 或者提高代码效率本身

除此之外，使用了编程语言的某种特性，如频繁的调用 JNI，反复性反射调用。除了通过积攒经验方式之外，通过工具解决的方法就是通过 CPU Profiler、simpleperf 等工具进行诊断

## 原因 3: 线程跑小核，导致执行时间长[](https://www.androidperformance.com/2022/03/13/android-systrace-cpu-state-running/#%E5%8E%9F%E5%9B%A0-3-%E7%BA%BF%E7%A8%8B%E8%B7%91%E5%B0%8F%E6%A0%B8%EF%BC%8C%E5%AF%BC%E8%87%B4%E6%89%A7%E8%A1%8C%E6%97%B6%E9%97%B4%E9%95%BF)

对 CPU Bound 的操作来说跑在小核可能没法满足性能需求，因为小核的定位是处理非UX 强相关的线程。不过 Android 没办法保证这一点，有时候任务就是会安排在小核上执行。

优化建议：线程绑核、SchedBoost 等操作，让线程跑尽量跑更高算力的核上，比如大核。有时候即使迁核了也不见效，这时候要看看频率是否拉得足够高，见“原因 4”

## 原因 4: 线程所跑的大核运行频率太低[](https://www.androidperformance.com/2022/03/13/android-systrace-cpu-state-running/#%E5%8E%9F%E5%9B%A0-4-%E7%BA%BF%E7%A8%8B%E6%89%80%E8%B7%91%E7%9A%84%E5%A4%A7%E6%A0%B8%E8%BF%90%E8%A1%8C%E9%A2%91%E7%8E%87%E5%A4%AA%E4%BD%8E)

优化建议：

1. 优化代码逻辑，主动降低运行负载，CPU 频率低也能流畅运行
2. 修改调度器升频相关的参数，让 CPU 根据负载提频更激进
3. 用平台提供的接口锁定 CPU 频率（俗称的「锁频」）

## 原因 5: 温升导致 CPU 关核、限频[](https://www.androidperformance.com/2022/03/13/android-systrace-cpu-state-running/#%E5%8E%9F%E5%9B%A0-5-%E6%B8%A9%E5%8D%87%E5%AF%BC%E8%87%B4-CPU-%E5%85%B3%E6%A0%B8%E3%80%81%E9%99%90%E9%A2%91)

优化建议:

手机因结构原因导致散热能力差或温升参数过于激进时，为了保护体验跟不烫伤人，几乎所有手机厂家的系统会限制 CPU 频率或者直接关核。排查思路是首先需要找到触发温升的原因。

温升的排查的第一步，首先要看是外因导致还是内因导致。外因是指是否由外部高温导致，如太阳底下，火炉边；往往夏天的时候导致手机发热的情况越严重

内因主要由 CPU、Modem、相机模组或者其他发热比较厉害的器件导致的。以 CPU 为例，如果后台某个线程吃满 CPU，那就首先要解决它。如果是前台应用负载高导致大电流消耗，同样道理，那就降低前台本身的负载。其他器件也是同样道理，首先要看是否是无意义的运行，其次是优化业务逻辑本身

除此之外，温升参数过于激进的话导致触发限频关核的概率也会提高，因此通过与竞品对比等方式调优温升参数本身来达到优化目的

## 原因 6: CPU 算力弱[](https://www.androidperformance.com/2022/03/13/android-systrace-cpu-state-running/#%E5%8E%9F%E5%9B%A0-6-CPU-%E7%AE%97%E5%8A%9B%E5%BC%B1)

优化建议:

ARM 处理器在相同频率下不同微架构的配置导致的性能差异是非常明显的，不同运行频率、L1/L2 Cache 的容量均能影响 CPU 的 MIPS（**Million Instructions Per Second**） 执行结果。

优化思路有两条:

1. 编译器参数
2. 优化代码逻辑

第一条比较难，大部分应用开发者来说也不太现实，系统厂商如华为，方舟编译器优化 JNI 的思路本质是不改应用代码情况下提高代码执行效率来达到性能上的提升

第二条可以通过 simpleperf 等工具，找到热点代码或者观察 CPU 行为后做进一步的改善，如:

- Cache miss 率过高导致执行耗时，就要优化内存访问相关逻辑
- 代码复杂指令过多导致耗时，就要优化代码逻辑，降低代码复杂度
- 设计好业务缓存，尽量提高缓存命中率，避免抖动（反复地申请与释放）

# 原文链接

1. [Systrace 线程 CPU 运行状态分析技巧 - Running 篇](https://www.androidperformance.com/2022/03/13/android-systrace-cpu-state-running/)

