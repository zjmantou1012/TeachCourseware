---
author: zjmantou
title: Android Systrace 12 基础知识 - CPU Info 解读
time: 2024-03-16 周六
tags:
  - Android
  - 资料
  - 性能优化
  - Systrace
  - Tools
---
本文是 Systrace 系列文章的第十二篇，主要是对 Systrace 中的 CPU 信息区域(Kernel)进行简单介绍，简单介绍了如何在 Systrace 中查看 Kernel 模块输出的 CPU 相关的信息，了解 CPU 频率、调度、锁频、锁核相关的信息

本系列的目的是通过 Systrace 这个工具，从另外一个角度来看待 Android 系统整体的运行，同时也从另外一个角度来对 Framework 进行学习。也许你看了很多讲 Framework 的文章，但是总是记不住代码，或者不清楚其运行的流程，也许从 Systrace 这个图形化的角度，你可以理解的更深入一些。

# CPU 区域图例

下面是高通骁龙 845 手机 Systrace 对应的 Kernel 中的 CPU Info 区域（底下的一些这里不讲，主要是讲 Kernel CPU 信息）

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161712922.png)

Systrace 中 CPU Info 一般在最上面，里面经常会用到的信息包括：

1. CPU 频率变化情况
2. 任务执行情况
3. 大小核的调度情况
4. CPU Boost 调度情况

总的来说，Systrace 中的 Kernel CPU Info 这里一般是看任务调度信息，查看是否是频率或者调度导致当前任务出现性能问题，举例如下：

1. 某个场景的任务执行比较慢，我们就可以查看是不是这个任务被调度到了小核？
2. 某个场景的任务执行比较慢，当前执行这个任务的 CPU 频率是不是不够？
3. 我的任务比较特殊，比如指纹解锁，能不能把我这个任务放到大核去跑？
4. 我这个场景对 CPU 要求很高，我能不能要求在我这个场景运行的时候，限制 CPU 最低频率？

与 CPU 运行信息相关的内容在 [Systrace 基础知识 – 分析 Systrace 预备知识](https://www.androidperformance.com/2019/07/23/Android-Systrace-Pre/) 这篇文章里面有详细的讲解，不熟悉的同学可以配合这篇文章一起食用

# 核心架构

简单来说目前的手机 CPU 按照核心数和架构来说，可以分为下面三类：

1. 非大小核架构
2. 大小核架构
3. 大中小核架构

目前的大部分 CPU 都是大小核架构，当然也有一些 CPU 是大中小核架构，比如高通骁龙 855\865，也有少部分 CPU 是非大小核架构

下面就来说说各种架构的区别，方便大家后续查看 Systrace

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161712706.png)

## 非大小核架构[](https://www.androidperformance.com/2019/12/21/Android-Systrace-CPU#%E9%9D%9E%E5%A4%A7%E5%B0%8F%E6%A0%B8%E6%9E%B6%E6%9E%84)

很早的机器 CPU 只有双核心或者四核心的时候，一般只有一种核心架构，也就是说这四个核心或者两个核心是同构的，相同的频率，相同的功耗，一起开启或者关闭；有些高通的中低端处理器也会使用同构的八核心处理器，比如高通骁龙 636

现在的大部分机器已经不使用非大小核的架构了

## 大小核架构[](https://www.androidperformance.com/2019/12/21/Android-Systrace-CPU#%E5%A4%A7%E5%B0%8F%E6%A0%B8%E6%9E%B6%E6%9E%84)

现在的 CPU 一般采用 8 核心，八个核心中，CPU 0-3 一般是小核心，CPU 4-7，如下图中 Systrace 中就是按照这个排列的

小核心一般来说主频低，功耗也低，使用的一般是 arm A5X 系列，比如高通骁龙 845，小核心是由四个 A55 (最高主频 1.8GHz ) 组成

大核心一般来说最高主频比较高，功耗相对来说也会比较高，使用的一般是 arm A7X 系列，比如高通骁龙 845，大核心就是由四个 A75（最高主频 2.8GHz）组成

下图就是 845 的 CPU

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161712450.png)

当然大小核架构中还有一些变种，比如高通骁龙 636 (4 小核 + 2 大核）或者高通骁龙 710 (6 小核 + 2 大核），宗旨还是不变，**大核心用来支持高负载场景，小核心用来日常使用，至于够不够用，就看你舍不舍得花银子，毕竟一分价钱一分货，高通爸爸也不是做福利的**

下面这些高通的主流大小核处理器的参数如下

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161713323.png)


## 大中小核架构[](https://www.androidperformance.com/2019/12/21/Android-Systrace-CPU#%E5%A4%A7%E4%B8%AD%E5%B0%8F%E6%A0%B8%E6%9E%B6%E6%9E%84)

部分 CPU 比较另辟蹊径，选择了大中小核的架构，比如高通骁龙 855 8 核 (1 个 A76 的大核+3 个 A76 的中核 + 4 个 A55 的小核）和之前的的 MTK X30 10 核 (2 个 A73 的大核 + 4 个 A53 的中核 + 4 个 A35 的小核）以及麒麟 980 8 核 (2 个 A76 的大核 + 2 个 A76 的中核 + 4 个 A55 的小核）

相比大小核架构，大中小核架构中的大核可以理解为超大核(高通称之为 Gold +) ，这个超大核的个数一般比较少(1-2 个)，主频一般会比较高，功耗相对也会高很多，这个是用来处理一些比较繁重的任务

下图是 855、845 和麒麟 980 的对比

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161713698.png)

顺带提一嘴，今年的高通骁龙 865 依然是大中小核的架构，大核和中核用的是 A77 架构,小核用的是 A55，大核和中核最高频率不一样，**大核只有一个，主频到 2.8GHz**，不知道 865 Plus 会不会搞到 3GHz

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161713509.png)


# 绑核

绑核，顾名思义就是**把某个任务绑定到某个或者某些核心上，来满足这个任务的性能需求**：

1. 任务本身负载比较高，需要在大核心上面才能满足时间要求
2. 任务本身不想被频繁切换，需要绑定在某一个核心上面
3. 任务本身不重要，对时间要求不高，可以绑定或者限制在小核心上面运行

上面是一些绑核的例子，目前 Android 中绑核操作一般是由系统来实现的，常用的有三种方法

## 配置 CPUset[](https://www.androidperformance.com/2019/12/21/Android-Systrace-CPU#%E9%85%8D%E7%BD%AE-CPUset)

使用 CPUset 子系统可以限制某一类的任务跑在特定的 CPU 或者 CPU 组里面，比如下面，Android 中会划分一些默认的 CPU 组，厂商可以针对不同的 CPU 架构进行定制，目前默认划分

1. system-background 一些低优先级的任务会被划分到这里，只能跑到小核心里面
2. foreground 前台进程
3. top-app 目前正在前台和用户交互的进程
4. background 后台进程
5. foreground/boost 前台 boost 进程，通常是用来联动的，现在已经没有用到了，之前的时候是应用启动的时候，会把所有 foreground 里面的进程都迁移到这个进程组里面

每个 CPU 架构对应的 CPUset 的配置都不一样，每个厂商也会有不同的策略在里面，比如下面就是一个 Google 官方默认的配置，各位也可以查看对应的节点来查看自己的 CPUset 组的配置

```shell
//官方默认配置  
write /dev/CPUset/top-app/CPUs 0-7  
write /dev/CPUset/foreground/CPUs 0-7  
write /dev/CPUset/foreground/boost/CPUs 4-7  
write /dev/CPUset/background/CPUs 0-7  
write /dev/CPUset/system-background/CPUs 0-3  
// 自己查看  
adb shell cat /dev/CPUset/top-app/CPUs  
0-7
```

对应的，可以在每个 CPUset 组的 tasks 节点下面看有哪些进程和线程是跑在这个组里面的

```shell
$ adb shell cat /dev/CPUset/top-app/tasks  
1687  
1689  
1690  
3559
```

需要注意每个任务跑在哪个组里面，是动态的，并不是一成不变的，有权限的进程就可以改

部分进程也可以在启动的时候就配置好跑到哪个进程里面，下面是 lmkd 的启动配置，writepid /dev/CPUset/system-background/tasks 这一句把自己安排到了 system-background 这个组里面

```shell
service lmkd /system/bin/lmkd  
class core  
user lmkd  
group lmkd system readproc  
capabilities DAC_OVERRIDE KILL IPC_LOCK SYS_NICE SYS_RESOURCE BLOCK_SUSPEND  
critical  
socket lmkd seqpacket 0660 system system  
writepid /dev/CPUset/system-background/tasks
```

大部分 App 进程是根据状态动态去变化的,在 Process 这个类中有详细的定义

android/os/Process.java

```java
/**  
 * Default thread group -  
 * has meaning with setProcessGroup() only, cannot be used with setThreadGroup().  
 * When used with setProcessGroup(), the group of each thread in the process  
 * is conditionally changed based on that thread's current priority, as follows:  
 * threads with priority numerically less than THREAD_PRIORITY_BACKGROUND  
 * are moved to foreground thread group.  All other threads are left unchanged.  
 * @hide  
 */  
public static final int THREAD_GROUP_DEFAULT = -1;  
  
/**  
 * Background thread group - All threads in  
 * this group are scheduled with a reduced share of the CPU.  
 * Value is same as constant SP_BACKGROUND of enum SchedPolicy.  
 * FIXME rename to THREAD_GROUP_BACKGROUND.  
 * @hide  
 */  
public static final int THREAD_GROUP_BG_NONINTERACTIVE = 0;  
  
/**  
 * Foreground thread group - All threads in  
 * this group are scheduled with a normal share of the CPU.  
 * Value is same as constant SP_FOREGROUND of enum SchedPolicy.  
 * Not used at this level.  
 * @hide  
 **/  
private static final int THREAD_GROUP_FOREGROUND = 1;  
  
/**  
 * System thread group.  
 * @hide  
 **/  
public static final int THREAD_GROUP_SYSTEM = 2;  
  
/**  
 * Application audio thread group.  
 * @hide  
 **/  
public static final int THREAD_GROUP_AUDIO_APP = 3;  
  
/**  
 * System audio thread group.  
 * @hide  
 **/  
public static final int THREAD_GROUP_AUDIO_SYS = 4;  
  
/**  
 * Thread group for top foreground app.  
 * @hide  
 **/  
public static final int THREAD_GROUP_TOP_APP = 5;  
  
/**  
 * Thread group for RT app.  
 * @hide  
 **/  
public static final int THREAD_GROUP_RT_APP = 6;  
  
/**  
 * Thread group for bound foreground services that should  
 * have additional CPU restrictions during screen off  
 * @hide  
 **/  
 public static final int THREAD_GROUP_RESTRICTED = 7;
```

在 OomAdjuster 中会动态根据进程的状态修改其对应的 CPUset 组， 详细可以自行查看 OomAdjuster 中 computeOomAdjLocked、updateOomAdjLocked、applyOomAdjLocked 的执行逻辑(Android 10)

## 配置 affinity[](https://www.androidperformance.com/2019/12/21/Android-Systrace-CPU#%E9%85%8D%E7%BD%AE-affinity)

使用 affinity 也可以设置任务跑在哪个核心上，其系统调用的 taskset， taskset 用来查看和设定“CPU 亲和力”，其实就是查看或者配置进程和 CPU 的绑定关系，让某进程在指定的 CPU 核上运行，即是“绑核”。

### taskset 的用法[](https://www.androidperformance.com/2019/12/21/Android-Systrace-CPU#taskset-%E7%9A%84%E7%94%A8%E6%B3%95)

**显示进程运行的CPU**

```shell
taskset -p pid
```

注意，此命令返回的是十六进制的，转换成二进制后，每一位对应一个逻辑 CPU，低位是 0 号CPU，依次类推。如果每个位置上是1，表示该进程绑定了该 CPU。例如，0101 就表示进程绑定在了 0 号和 3 号逻辑 CPU 上了

**绑核设定**

```shell
taskset -pc 3  pid    表示将进程pid绑定到第3个核上  
taskset -c 3 command   表示执行 command 命令，并将 command 启动的进程绑定到第3个核上。
```

Android 中也可以使用这个系统调用，把任务绑定到某个核心上运行。部分较老的内核里面不支持 CPUset，就会用 taskset 来设置

## 调度算法[](https://www.androidperformance.com/2019/12/21/Android-Systrace-CPU#%E8%B0%83%E5%BA%A6%E7%AE%97%E6%B3%95)

在 Linux 的调度算法中修改调度逻辑，也可以让指定的 task 跑在指定的核上面，部分厂家的核调度优化就是使用的这种方法，这里就不具体来讲了

# 锁频

正常情况下，CPU 的调度算法都可以满足日常的使用，但是在 Android 中的部分场景里面，单纯依靠调度器，可能会无法满足这个场景对性能的要求。比如说应用启动场景，如果让调度器去拉频率迁核，可能就会有一定的延迟，比如任务先在小核跑，发现小核频率不够，那就把小核频率往上拉，拉上去之后发现可能还是不够，经过几次一直拉到最高发现还是不够，然后把这个任务迁移到中核，频率也是一次一次拉，拉到最高发现还是不够，最好迁移到大核去做。这样一套下来，时间过去不少不说，启动速度也不是最快的

基于这种情况的考虑，系统中一般都会在这种特殊场景直接暴力拉核，将硬件资源直接拉到最高去运行，比如 CPU、GPU、IO、BUS 等；另外也会在某些场景把某些资源限制使用，比如发热太严重的时候，需要限制 CPU 的最高频率，来达到降温的目的；有时候基于功耗的考虑，也会限制一些资源在某些场景的使用

目前 Android 系统一般会在下面几个场景直接进行锁频（不同厂家也会自己定制）

1. 应用启动
2. 应用安装
3. 转屏
4. 窗口动画
5. List Fling
6. Game

以 高通平台为例，在 CPU Info 中我们也可以看到锁频的情况

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161716745.png)
# CPU 状态

CPU info 中还有标识 CPU 状态的标记，如下图所示，CPU 状态有 0 ，1，2，3 这四种

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161716916.png)

之前的 CPU 支持热插拔，即不用的时候可以直接关闭，不过目前的 CPU 都不支持热插拔，而是使用 C-State

下面是摘抄的其他平台的支持 C0-C4 的处理器的状态和功耗状态，Android 中不同的平台表现不一致，大家可以做一下参考

1. C0 状态（激活）
2. 这是 CPU 最大工作状态，在此状态下可以接收指令和处理数据
3. 所有现代处理器必须支持这一功耗状态
4. C1 状态（挂起）
5. 可以通过执行汇编指令“ HLT （挂起）”进入这一状态
6. 唤醒时间超快！（快到只需 10 纳秒！）
7. 可以节省 70% 的 CPU 功耗
8. 所有现代处理器都必须支持这一功耗状态
9. C2 状态（停止允许）
10. 处理器时钟频率和 I/O 缓冲被停止
11. 换言之，处理器执行引擎和 I/0 缓冲已经没有时钟频率
12. 在 C2 状态下也可以节约 70% 的 CPU 和平台能耗
13. 从 C2 切换到 C0 状态需要 100 纳秒以上
14. C3 状态（深度睡眠）
15. 总线频率和 PLL 均被锁定
16. 在多核心系统下，缓存无效
17. 在单核心系统下，内存被关闭，但缓存仍有效可以节省 70% 的 CPU 功耗，但平台功耗比 C2 状态下大一些
18. 唤醒时间需要 50 微妙

# Systrace 中的详细信息

Systrace 我们一般用 Chrome 打开，转换成图形化信息之后更加方便从整体去看，但其实 Systrace 也可以以文本的方式打开，也可以看到一些详细的信息。

比如下面就是一条标识 CPU 调度的 Message，解析的时候，里面的信息会被解析到各个模块

```shell
appEventThread-8193  [001] d..2 1638545.400415: sched_switch: prev_comm=appEventThread prev_pid=8193 prev_prio=97 prev_state=S ==> next_comm=swapper/1 next_pid=0 next_prio=120
```

详细来看

```shell
appEventThread-8193 -- 标识 TASK-PID  
[001] -- 标识是哪个 CPU ，这里是 cpu0  
d..2 -- 这是四个位，每个位分别对应 irqs-off、need-resched、hardirq/softirq、preempt-depth  
1638545.400415 -- 标识 delay TIMESTAMP  
sched_switch ...到最后 -- 标识信息区，里面包含前一个任务描述，前一个任务的 pid，前一个任务的优先级 ，当前任务，当前任务 pid，当前任务优先级
```

另外里面仔细看也可以看到许多有趣的输出，可以加深对调度的理解

1. sched_waking: comm=kworker/u16:4 pid=17373 prio=120 target_cpu=003
2. sched_blocked_reason: pid=17373 iowait=0 caller=rpmh_write_batch+0x638/0x7d0
3. cpu_idle: state=0 cpu_id=3
4. softirq_raise: vec=6 [action=TASKLET]
5. cpu_frequency_limits: min=1555200 max=1785600 cpu_id=0
6. cpu_frequency_limits: min=710400 max=2419200 cpu_id=4
7. cpu_frequency_limits: min=825600 max=2841600 cpu_id=7

# 参考

1. [绑定CPU逻辑核心的利器——taskset](https://blog.csdn.net/breaksoftware/article/details/79160916)
2. [CPU 电源状态](http://www.voidcn.com/article/p-kcjkqmld-bmg.html)

1. [Systrace 基础知识 - CPU Info 解读](https://www.androidperformance.com/2019/12/21/Android-Systrace-CPU)
