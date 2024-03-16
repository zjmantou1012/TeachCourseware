---
author: zjmantou
title: Android Systrace 10 基础知识 - Binder 和锁竞争解读
time: 2024-03-16 周六
tags:
  - Android
  - 资料
  - 性能优化
  - Systrace
  - Tools
---
本文是 Systrace 系列文章的第十篇，主要是对 Systrace 中的 Binder 和锁信息进行简单介绍，简单介绍了 Binder 的情况，介绍了 Systrace 中 Binder 通信的表现形式，以及 Binder 信息查看，SystemServer 锁竞争分析等

本系列的目的是通过 Systrace 这个工具，从另外一个角度来看待 Android 系统整体的运行，同时也从另外一个角度来对 Framework 进行学习。也许你看了很多讲 Framework 的文章，但是总是记不住代码，或者不清楚其运行的流程，也许从 Systrace 这个图形化的角度，你可以理解的更深入一些。

# Binder 概述

Android 的大部分进程间通信都使用 Binder，这里对 Binder 不做过多的解释，想对 Binder 的实现有一个比较深入的了解的话，推荐你阅读下面三篇文章

1. [理解Android Binder机制1/3：驱动篇](https://paul.pub/android-binder-driver/)
2. [理解Android Binder机制2/3：C++层](https://paul.pub/android-binder-cpp/)
3. [理解Android Binder机制3/3：Java层](https://paul.pub/android-binder-java/)

**之所以要单独讲 Systrace 中的 Binder 和锁，是因为很多卡顿问题和响应速度的问题，是因为跨进程 binder 通信的时候，锁竞争导致 binder 通信事件变长，影响了调用端。最常见的就是应用渲染线程 dequeueBuffer 的时候 SurfaceFlinger 主线程阻塞导致 dequeueBuffer 耗时，从而导致应用渲染出现卡顿; 或者 SystemServer 中的 AMS 或者 WMS 持锁方法等待太多, 导致应用调用的时候等待时间比较长导致主线程卡顿**

这里放一张文章里面的 Binder 架构图 ， 本文主要是以 Systrace 为主，所以会讲 Systrace 中的 Binder 表现，不涉及 Binder 的实现

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161642696.png)


# Binder 调用图例

Binder 主要是用来跨进程进行通信，可以看下面这张图，简单显示了在 Systrace 中 ，Binder 通信是如何显示的

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161642502.png)

图中主要是 SystemServer 进程和 高通的 perf 进程通信，Systrace 中右上角 ViewOption 里面勾选 Flow Events 就可以看到 Binder 的信息

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161642065.png)


点击 Binder 可以查看其详细信息，其中有的信息在分析问题的时候可以用到，这里不做过多的描述

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161643872.png)

对于 Binder，这里主要介绍如何在 Systrace 中查看 Binder **锁信息**和**锁等待**这两个部分，很多卡顿和响应问题的分析，都离不开这两部分信息的解读，不过最后还是要回归代码，找到问题后，要读源码来理顺其代码逻辑，以方便做相应的优化工作

# Systrace 显示的锁的信息

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161643738.png)

**monitor contention with owner Binder:1605_B (4667) at void com.android.server.wm.ActivityTaskManagerService.activityPaused(android.os.IBinder)(ActivityTaskManagerService.java:1733) waiters=2 blocking from android.app.ActivityManager$StackInfo com.android.server.wm.ActivityTaskManagerService.getFocusedStackInfo()(ActivityTaskManagerService.java:2064)**

上面的话分两段来看，以 **blocking** 为分界线　

## 第一段信息解读[](https://www.androidperformance.com/2019/12/06/Android-Systrace-Binder/#%E7%AC%AC%E4%B8%80%E6%AE%B5%E4%BF%A1%E6%81%AF%E8%A7%A3%E8%AF%BB)

**monitor contention with owner Binder:1605_B (4667) at void com.android.server.wm.ActivityTaskManagerService.activityPaused(android.os.IBinder)(ActivityTaskManagerService.java:1733) waiters=2**

**Monitor** 指的是当前锁对象的池，在 Java 中，每个对象都有两个池，锁(monitor)池和等待池：

**锁池**（同步队列 SynchronizedQueue ）：假设线程 A 已经拥有了某个对象(注意:不是类 )的锁，而其它的线程想要调用这个对象的某个 synchronized 方法(或者 synchronized 块)，由于这些线程在进入对象的 synchronized 方法之前必须先获得该对象的锁的拥有权，但是该对象的锁目前正被线程 A 拥有，所以这些线程就进入了该对象的锁池中。

这里用了争夺(contention)这个词，意思是这里由于在和目前对象的锁正被其他对象（Owner）所持有，所以没法得到该对象的锁的拥有权，所以进入该对象的锁池

**Owner** : 指的是当前**拥有**这个对象的锁的对象。这里是 Binder:1605_B，4667 是其线程 ID。

**at** 后面跟的是**拥有**这个对象的锁的对象正在做什么。这里是在执行 void com.android.server.wm.ActivityTaskManagerService.activityPaused 这个方法，其代码位置是 ：ActivityTaskManagerService.java:1733 其对应的代码如下：

com/android/server/wm/ActivityTaskManagerService.java

```java
@Override  
public final void activityPaused(IBinder token) {  
    final long origId = Binder.clearCallingIdentity();  
    synchronized (mGlobalLock) { // 1733 是这一行  
        ActivityStack stack = ActivityRecord.getStackLocked(token);  
        if (stack != null) {  
            stack.activityPausedLocked(token, false);  
        }  
    }  
    Binder.restoreCallingIdentity(origId);  
}
```

可以看到这里 synchronized (mGlobalLock) ，获取了 mGlobalLock 锁的拥有权，在他释放这个对象的锁之前，任何其他的调用 synchronized (mGlobalLock) 的地方都得在锁池中等待

**waiters** 值得是锁池里面正在等待锁的操作的个数；这里 waiters=2 表示目前锁池里面已经有一个操作在等待这个对象的锁释放了，加上这个的话就是 3 个了

## 第二段信息解读[](https://www.androidperformance.com/2019/12/06/Android-Systrace-Binder/#%E7%AC%AC%E4%BA%8C%E6%AE%B5%E4%BF%A1%E6%81%AF%E8%A7%A3%E8%AF%BB)

**blocking from android.app.ActivityManager$StackInfo com.android.server.wm.ActivityTaskManagerService.getFocusedStackInfo()(ActivityTaskManagerService.java:2064)**

第二段信息相对来说简单一些，就是标识了当前被阻塞等锁的方法 ， 这里是 ActivityManager 的 getFocusedStackInfo 被阻塞，其对应的代码

com/android/server/wm/ActivityTaskManagerService.java

```java
@Override  
public ActivityManager.StackInfo getFocusedStackInfo() throws RemoteException {  
    enforceCallerIsRecentsOrHasPermission(MANAGE_ACTIVITY_STACKS, "getStackInfo()");  
    long ident = Binder.clearCallingIdentity();  
    try {  
        synchronized (mGlobalLock) { // 2064 是这一行   
            ActivityStack focusedStack = getTopDisplayFocusedStack();  
            if (focusedStack != null) {  
                return mRootActivityContainer.getStackInfo(focusedStack.mStackId);  
            }  
            return null;  
        }  
    } finally {  
        Binder.restoreCallingIdentity(ident);  
    }  
}
```

待获取 ams 对象的锁拥有权

## 总结[](https://www.androidperformance.com/2019/12/06/Android-Systrace-Binder/#%E6%80%BB%E7%BB%93)

上面这段话翻译过来就是

**ActivityTaskManagerService 的 getFocusedStackInfo 方法在执行过程中被阻塞，原因是因为执行同步方法块的时候，没有拿到同步对象的锁的拥有权；需要等待拥有同步对象的锁拥有权的另外一个方法 ActivityTaskManagerService.activityPaused 执行完成后，才能拿到同步对象的锁的拥有权，然后继续执行**

可以对照原文看上面的翻译

**monitor contention with owner Binder:1605_B (4667)  
at void com.android.server.wm.ActivityTaskManagerService.activityPaused(android.os.IBinder)(ActivityTaskManagerService.java:1733)  
waiters=2  
blocking from android.app.ActivityManager$StackInfo com.android.server.wm.ActivityTaskManagerService.getFocusedStackInfo()(ActivityTaskManagerService.java:2064)**

# 等锁分析

还是上面那个 Systrace，Binder 信息里面显示 waiters=2，意味着前面还有两个操作在等锁释放，也就是说总共有三个操作都在等待 Binder:1605_B (4667) 释放锁，我们来看一下 Binder:1605_B 的执行情况

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161644157.png)
从上图可以看到，Binder:1605_B 正在执行 activityPaused，中间也有一些其他的 Binder 操作，最终 activityPaused 执行完成后，释放锁

下面我们就把这个逻辑里面的执行顺序理顺，包括两个 **waiters**

## 锁等待


![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161644910.png)


上图中可以看到 mGlobalLock 这个对象锁的争夺情况

1. Binder_1605_B 首先开始执行 **activityPaused**，这个方法中是要获取 mGlobalLock 对象锁的，由于此时 mGlobalLock 没有竞争，所以 activityPaused 获取对象锁之后开始执行
2. android.display 线程开始执行 **checkVisibility** 方法，这个方法也是要获取 mGlobalLock 对象锁的，但是此时 Binder_1605_B 的 activityPaused 持有 mGlobalLock 对象锁 ，所以这里 android.display 的 checkVisibility 开始等待，进入 sleep 状态
3. android.anim 线程开始执行 **relayoutWindow** 方法，这个方法也是要获取 mGlobalLock 对象锁的，但是此时 Binder_1605_B 的 activityPaused 持有 mGlobalLock 对象锁 ，所以这里 android.display 的 checkVisibility 开始等待，进入 sleep 状态
4. android.bg 线程开始执行 **getFocusedStackInfo** 方法，这个方法也是要获取 mGlobalLock 对象锁的，但是此时 Binder_1605_B 的 activityPaused 持有 mGlobalLock 对象锁 ，所以这里 android.display 的 checkVisibility 开始等待，进入 sleep 状态

经过上面四步，就形成了 Binder_1605_B 线程在运行，其他三个争夺 mGlobalLock 对象锁失败的线程分别进入 sleep 状态，等待 Binder_1605_B 执行结束后释放 mGlobalLock 对象锁

## 锁释放

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161645119.png)


上图可以看到 mGlobalLock 锁的释放和后续的流程

1. Binder_1605_B 线程的 **activityPaused** 执行结束，mGlobalLock 对象锁释放
2. 第一个进入等待的 android.display 线程开始执行 **checkVisibility** 方法 ，这里从 android.display 线程的唤醒信息可以看到，是被 Binder_1605_B(4667) 唤醒的
3. android.display 线程的 **checkVisibility** 执行结束，mGlobalLock 对象锁释放
4. 第二个进入等待的 android.anim 线程开始执行 **relayoutWindow** 方法 ，这里从 android.anim 线程的唤醒信息可以看到，是被 android.display(1683) 唤醒的
5. android.anim 线程的 **relayoutWindow** 执行结束，mGlobalLock 对象锁释放
6. 第三个进入等待的 android.bg 线程开始执行 **getFocusedStackInfo** 方法 ，这里从 android.bg 线程的唤醒信息可以看到，是被 android.anim(1684) 唤醒的

经过上面 6 步，这一轮由于 mGlobalLock 对象锁引起的等锁现象结束。这里只是一个简单的例子，在实际情况下，SystemServer 中的 BInder 等锁情况会非常严重，经常 waiter 会到达 7 - 10 个，非常恐怖，比如下面这种：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161646220.png)


这也就可以解释为什么 Android 手机 App 安装多了、用的久了之后，系统就会卡的一个原因；另外重启后也会有短暂的时候出现这种情况

如果不知道怎么查看唤醒信息，可以查看： [Systrace中查看进程信息唤醒](https://www.androidperformance.com/2019/07/23/Android-Systrace-Pre/#%E8%BF%9B%E7%A8%8B%E5%94%A4%E9%86%92%E4%BF%A1%E6%81%AF%E5%88%86%E6%9E%90) 这篇文章

# 相关代码

### Monitor 信息[](https://www.androidperformance.com/2019/12/06/Android-Systrace-Binder/#Monitor-%E4%BF%A1%E6%81%AF)

art/runtime/monitor.cc

```c++
std::string Monitor::PrettyContentionInfo(const std::string& owner_name,  
                                          pid_t owner_tid,  
                                          ArtMethod* owners_method,  
                                          uint32_t owners_dex_pc,  
                                          size_t num_waiters) {  
  Locks::mutator_lock_->AssertSharedHeld(Thread::Current());  
  const char* owners_filename;  
  int32_t owners_line_number = 0;  
  if (owners_method != nullptr) {  
    TranslateLocation(owners_method, owners_dex_pc, &owners_filename, &owners_line_number);  
  }  
  std::ostringstream oss;  
  oss << "monitor contention with owner " << owner_name << " (" << owner_tid << ")";  
  if (owners_method != nullptr) {  
    oss << " at " << owners_method->PrettyMethod();  
    oss << "(" << owners_filename << ":" << owners_line_number << ")";  
  }  
  oss << " waiters=" << num_waiters;  
  return oss.str();  
}
```

### Block 信息[](https://www.androidperformance.com/2019/12/06/Android-Systrace-Binder/#Block-%E4%BF%A1%E6%81%AF)

art/runtime/monitor.cc

```c++
if (ATRACE_ENABLED()) {  
  if (owner_ != nullptr) {  // Did the owner_ give the lock up?  
    std::ostringstream oss;  
    std::string name;  
    owner_->GetThreadName(name);  
    oss << PrettyContentionInfo(name,  
                                owner_->GetTid(),  
                                owners_method,  
                                owners_dex_pc,  
                                num_waiters);  
    // Add info for contending thread.  
    uint32_t pc;  
    ArtMethod* m = self->GetCurrentMethod(&pc);  
    const char* filename;  
    int32_t line_number;  
    TranslateLocation(m, pc, &filename, &line_number);  
    oss << " blocking from "  
        << ArtMethod::PrettyMethod(m) << "(" << (filename != nullptr ? filename : "null")  
        << ":" << line_number << ")";  
    ATRACE_BEGIN(oss.str().c_str());  
    started_trace = true;  
  }  
}
```

# 参考

1. [理解Android Binder机制1/3：驱动篇](https://paul.pub/android-binder-driver/)
2. [理解Android Binder机制2/3：C++层](https://paul.pub/android-binder-cpp/)
3. [理解Android Binder机制3/3：Java层](https://paul.pub/android-binder-java/)

# 附件

本文涉及到的附件也上传了，各位下载后解压，使用 **Chrome** 浏览器打开即可  
[点此链接下载文章所涉及到的 Systrace 附件](https://github.com/Gracker/SystraceForBlog/tree/master/Android_Systrace_Binder)

# 原文链接

1. [Systrace 基础知识 - Binder 和锁竞争解读](https://www.androidperformance.com/2019/12/06/Android-Systrace-Binder/)
