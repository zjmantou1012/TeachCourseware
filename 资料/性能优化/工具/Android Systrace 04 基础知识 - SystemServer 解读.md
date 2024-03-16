---
author: zjmantou
title: Android Systrace 04 基础知识 - SystemServer 解读
time: 2024-03-16 周六
tags:
  - Android
  - 性能优化
  - 资料
  - Systrace
  - Tools
---
本文是 Systrace 系列文章的第四篇，主要是对 SystemServer 进行简单介绍，介绍了 SystemServer 中几个比较重要的线程，由于 Input 和 Binder 比较重要，所以单独拿出来讲，在这里就没有再涉及到。

本系列的目的是通过 Systrace 这个工具，从另外一个角度来看待 Android 系统整体的运行，同时也从另外一个角度来对 Framework 进行学习。也许你看了很多讲 Framework 的文章，但是总是记不住代码，或者不清楚其运行的流程，也许从 Systrace 这个图形化的角度，你可以理解的更深入一些。

# 正文

## 窗口动画[](https://www.androidperformance.com/2019/06/29/Android-Systrace-SystemServer/#%E7%AA%97%E5%8F%A3%E5%8A%A8%E7%94%BB)

Systrace 中的 SystemServer 一个比较重要的地方就是窗口动画，由于窗口归 SystemServer 来管，那么窗口动画也就是由 SystemServer 来进行统一的处理，其中涉及到两个比较重要的线程，Android.Anim 和 Android.Anim.if 这两个线程，这两个线程的基本知识在下面有讲。

这里我们以**应用启动**为例，查看窗口时如何在两个线程之间进行切换(Android P 里面，应用的启动动画由 Launcher 和应用自己的第一帧组成，之前是在 SystemServer 里面的，现在多任务的动画为了性能部分移到了 Launcher 去实现)

首先我们点击图标启动应用的时候，由于 App 还在启动，Launcher 首先启动一个 StartingWindow，等 App 的第一帧绘制好了之后，再切换到 App 的窗口动画

Launcher 动画

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161457145.png)


此时对应的，App 正在启动

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161457392.png)

从上图可以看到，应用第一帧已经准备好了，接下来看对应的 SystemServer ，可以看到应用启动第一帧绘制完成后，动画切换到 App 的 Window 动画

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161457540.png)

## ActivityManagerService[](https://www.androidperformance.com/2019/06/29/Android-Systrace-SystemServer/#ActivityManagerService)

AMS 和 WMS 算是 SystemServer 中最繁忙的两个 Service 了，与 AMS 相关的 Trace 一般会用 TRACE_TAG_ACTIVITY_MANAGER 这个 TAG，在 Systrace 中的名字是 ActivityManager

下面是启动一个新的进程的时候，AMS 的输出

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161457370.png)

在进程和四大组件的各种场景一般都会有对应的 Trace 点来记录，比如大家熟悉的 ActivityStart、ActivityResume、activityStop 等，这些 Trace 点有一些在应用进程，有一些在 SystemServer 进程，所以大家在看 Activity 相关的代码逻辑的时候，需要不断在这两个进程之间进行切换，这样才能从一个整体的角度来看应用的状态变化和 SystemServer 在其中起到的作用。

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161458412.png)

## WindowManagerService[](https://www.androidperformance.com/2019/06/29/Android-Systrace-SystemServer/#WindowManagerService)

与 WMS 相关的 Trace 一般会用 TRACE_TAG_WINDOW_MANAGER 这个 TAG，在 Systrace 中 WindowManagerService 在 SystemServer 中多在对应的 Binder 中出现，比如下面应用启动的时候，relayoutWindow 的 Trace 输出

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161458743.png)

在 Window 的各种场景一般都会有对应的 Trace 点来记录，比如大家熟悉的 relayoutWIndow、performLayout、prepareToDisplay 等

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161458262.png)

## Input[](https://www.androidperformance.com/2019/06/29/Android-Systrace-SystemServer/#Input)

Input 是 SystemServer 线程里面非常重要的一部分，主要是由 InputReader 和 InputDispatcher 这两个 Native 线程组成，关于这一部分在 [Systrace 基础知识 - Input 解读](https://www.androidperformance.com/2019/11/04/Android-Systrace-Input/) 里面已经详细讲过，这里就不再详细讲了

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161458784.png)


## Binder[](https://www.androidperformance.com/2019/06/29/Android-Systrace-SystemServer/#Binder)

SystemServer 由于提供大量的基础服务，所以进程间的通信非常繁忙，且大部分通信都是通过 Binder ，所以 Binder 在 SystemServer 中的作用非常关键，很多时候当后台有大量的 App 存在的时候，SystemServer 就会由于 Binder 通信和锁竞争，导致系统或者 App 卡顿。关于这一部分在 [Binder 和锁竞争解读](https://www.androidperformance.com/2019/12/06/Android-Systrace-Binder/) 里面已经详细讲过，这里就不再详细讲了

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161459271.png)

## HandlerThread[](https://www.androidperformance.com/2019/06/29/Android-Systrace-SystemServer/#HandlerThread)

### BackgroundThread[](https://www.androidperformance.com/2019/06/29/Android-Systrace-SystemServer/#BackgroundThread)

com/android/internal/os/BackgroundThread.java

```Java
private BackgroundThread() {
    super("android.bg", android.os.Process.THREAD_PRIORITY_BACKGROUND);
}
```

Systrace 中的 BackgroundThread 

![](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/20240316150027.png)

BackgroundThread 在系统中使用比较多，许多对性能没有要求的任务，一般都会放到 BackgroundThread 中去执行

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161500345.png)

## ServiceThread[](https://www.androidperformance.com/2019/06/29/Android-Systrace-SystemServer/#ServiceThread)

ServiceThread 继承自 HandlerThread ，下面介绍的几个工作线程都是继承自 ServiceThread ，分别实现不同的功能，根据线程功能不同，其线程优先级也不同：UIThread、IoThread、DisplayThread、AnimationThread、FgThread、SurfaceAnimationThread

每个 Thread 都有自己的 Looper 、Thread 和 MessageQueue，互相不会影响。Android 系统根据功能，会使用不同的 Thread 来完成。

### UiThread[](https://www.androidperformance.com/2019/06/29/Android-Systrace-SystemServer/#UiThread)

com/android/server/UiThread.java

```java
private UiThread() {  
    super("android.ui", Process.THREAD_PRIORITY_FOREGROUND, false /*allowIo*/);  
}
```

Systrace 中的 UiThread

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161501462.png)


UiThread 被使用的地方如下，具体的功能可以自己去源码里面查看，关键字是 UiThread.get()

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161501036.png)


### IoThread[](https://www.androidperformance.com/2019/06/29/Android-Systrace-SystemServer/#IoThread)

com/android/server/IoThread.java

```java
private IoThread() {  
    super("android.io", android.os.Process.THREAD_PRIORITY_DEFAULT, true /*allowIo*/);  
}
```

IoThread 被使用的地方如下，具体的功能可以自己去源码里面查看，关键字是 IoThread.get()

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161502116.png)


### DisplayThread[](https://www.androidperformance.com/2019/06/29/Android-Systrace-SystemServer/#DisplayThread)

com/android/server/DisplayThread.java

```java
private DisplayThread() {  
    // DisplayThread runs important stuff, but these are not as important as things running in  
    // AnimationThread. Thus, set the priority to one lower.  
    super("android.display", Process.THREAD_PRIORITY_DISPLAY + 1, false /*allowIo*/);  
}
```

Systrace 中的 DisplayThread

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161503173.png)

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161503674.png)


### AnimationThread[](https://www.androidperformance.com/2019/06/29/Android-Systrace-SystemServer/#AnimationThread)

com/android/server/AnimationThread.java

```java
private AnimationThread() {  
    super("android.anim", THREAD_PRIORITY_DISPLAY, false /*allowIo*/);  
}
```

Systrace 中的 AnimationThread

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161503154.png)

AnimationThread 在源码中的使用，可以看到 WindowAnimator 的动画执行也是在 AnimationThread 线程中的，Android P 增加了一个 SurfaceAnimationThread 来分担 AnimationThread 的部分工作，来提高 WindowAnimation 的动画性能

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161504371.png)


### FgThread[](https://www.androidperformance.com/2019/06/29/Android-Systrace-SystemServer/#FgThread)

com/android/server/FgThread.java

```java
private FgThread() {  
    super("android.fg", android.os.Process.THREAD_PRIORITY_DEFAULT, true /*allowIo*/);  
}
```

Systrace 中的 FgThread

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161504214.png)


FgThread 在源码中的使用，可以自己搜一下，下面是具体的使用的一个例子

```java
FgThread.getHandler().post(() -> {  
    synchronized (mLock) {  
        if (mStartedUsers.get(userIdToLockF) != null) {  
            Slog.w(TAG, "User was restarted, skipping key eviction");  
            return;  
        }  
    }  
    try {  
        mInjector.getStorageManager().lockUserKey(userIdToLockF);  
    } catch (RemoteException re) {  
        throw re.rethrowAsRuntimeException();  
    }  
    if (userIdToLockF == userId) {  
        for (final KeyEvictedCallback callback : keyEvictedCallbacks) {  
            callback.keyEvicted(userId);  
        }  
    }  
});
```

### SurfaceAnimationThread

com/android/server/wm/SurfaceAnimationThread.java  

```java
private SurfaceAnimationThread() {  
    super("android.anim.lf", THREAD_PRIORITY_DISPLAY, false /*allowIo*/);  
}
```

Systrace 中的 SurfaceAnimationThread 

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161505692.png)

SurfaceAnimationThread 的名字叫 android.anim.lf ， 与 android.anim 有区别，  

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161506793.png)


这个 Thread 主要是执行窗口动画，用于分担 android.anim 线程的一部分动画工作，减少由于锁导致的窗口动画卡顿问题，具体的内容可以看这篇文章：[Android P——LockFreeAnimation](https://zhuanlan.zhihu.com/p/44864987)

```java
SurfaceAnimationRunner(@Nullable AnimationFrameCallbackProvider callbackProvider,  
        AnimatorFactory animatorFactory, Transaction frameTransaction,  
        PowerManagerInternal powerManagerInternal) {  
    SurfaceAnimationThread.getHandler().runWithScissors(() -> mChoreographer = getSfInstance(),  
            0 /* timeout */);  
    mFrameTransaction = frameTransaction;  
    mAnimationHandler = new AnimationHandler();  
    mAnimationHandler.setProvider(callbackProvider != null  
            ? callbackProvider  
            : new SfVsyncFrameCallbackProvider(mChoreographer));  
    mAnimatorFactory = animatorFactory != null  
            ? animatorFactory  
            : SfValueAnimator::new;  
    mPowerManagerInternal = powerManagerInternal;  
}
```

# 原文链接

[Systrace 基础知识 - SystemServer 解读](https://www.androidperformance.com/2019/06/29/Android-Systrace-SystemServer/)

