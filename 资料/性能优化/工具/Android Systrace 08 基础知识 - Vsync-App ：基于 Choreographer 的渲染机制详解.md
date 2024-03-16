---
author: zjmantou
title: Android Systrace 08 基础知识 - Vsync-App ：基于 Choreographer 的渲染机制详解
time: 2024-03-16 周六
tags:
  - Android
  - 资料
  - 性能优化
  - Systrace
  - Tools
---
本文介绍了 App 开发者不经常接触到但是在 Android Framework 渲染链路中非常重要的一个类 Choreographer。包括 Choreographer 的引入背景、Choreographer 的简介、部分源码解析、Choreographer 与 MessageQueue、Choreographer 和 APM，以及手机厂商基于 Choreographer 的一些优化思路

Choreographer 的引入，主要是配合 Vsync ，给上层 App 的渲染提供一个稳定的 Message 处理的时机，也就是 Vsync 到来的时候 ，系统通过对 Vsync 信号周期的调整，来控制每一帧绘制操作的时机. 目前大部分手机都是 60Hz 的刷新率，也就是 16.6ms 刷新一次，系统为了配合屏幕的刷新频率，将 Vsync 的周期也设置为 16.6 ms，每个 16.6 ms ， Vsync 信号唤醒 Choreographer 来做 App 的绘制操作 ，这就是引入 Choreographer 的主要作用. 了解 Choreographer 还可以帮助 App 开发者知道程序每一帧运行的基本原理，也可以加深对 Message、Handler、Looper、MessageQueue、Measure、Layout、Draw 的理解

本文是 Systrace 系列文章的第八篇，主要是对 Systrace 中的 Choreographer 进行简单介绍

本系列的**目的**是通过 Systrace 这个工具，从另外一个角度来看待 Android 系统整体的运行，同时也从另外一个角度来对 Framework 进行学习。也许你看了很多讲 Framework 的文章，但是总是记不住代码，或者不清楚其运行的流程，也许从 Systrace 这个图形化的角度，你可以理解的更深入一些。

# 主线程运行机制的本质

在讲 Choreographer 之前，我们先理一下 Android 主线程运行的本质，其实就是 Message 的处理过程，我们的各种操作，包括每一帧的渲染操作 ，都是通过 Message 的形式发给主线程的 MessageQueue ，MessageQueue 处理完消息继续等下一个消息，如下图所示

**MethodTrace 图示**

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161604759.png)


**Systrace 图示**

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161604610.png)

## 演进[](https://androidperformance.com/2019/10/22/Android-Choreographer/#%E6%BC%94%E8%BF%9B)

引入 Vsync 之前的 Android 版本，渲染一帧相关的 Message ，中间是没有间隔的，上一帧绘制完，下一帧的 Message 紧接着就开始被处理。这样的问题就是，帧率不稳定，可能高也可能低，不稳定，如下图

**MethodTrace 图示**

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161604810.png)

**Systrace 图示**

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161605103.png)

可以看到这时候的瓶颈是在 dequeueBuffer, 因为屏幕是有刷新周期的, FB 消耗 Front Buffer 的速度是一定的, 所以 SF 消耗 App Buffer 的速度也是一定的, 所以 App 会卡在 dequeueBuffer 这里,这就会导致 App Buffer 获取不稳定, 很容易就会出现卡顿掉帧的情况.

对于用户来说，稳定的帧率才是好的体验，比如你玩王者荣耀，相比 fps 在 60 和 40 之间频繁变化，用户感觉更好的是稳定在 50 fps 的情况.

所以 Android 的演进中，引入了 **Vsync + TripleBuffer + Choreographer** 的机制，其主要目的就是提供一个稳定的帧率输出机制，让软件层和硬件层可以以共同的频率一起工作。

## 引入 Choreographer[](https://androidperformance.com/2019/10/22/Android-Choreographer/#%E5%BC%95%E5%85%A5-Choreographer)

Choreographer 的引入，主要是配合 Vsync ，给上层 App 的渲染提供一个稳定的 Message 处理的时机，也就是 Vsync 到来的时候 ，系统通过对 Vsync 信号周期的调整，来控制每一帧绘制操作的时机. 至于为什么 Vsync 周期选择是 16.6ms (60 fps) ，是因为目前大部分手机的屏幕都是 60Hz 的刷新率，也就是 16.6ms 刷新一次，系统为了配合屏幕的刷新频率，将 Vsync 的周期也设置为 16.6 ms，每隔 16.6 ms ，Vsync 信号到来唤醒 Choreographer 来做 App 的绘制操作 ，如果每个 Vsync 周期应用都能渲染完成，那么应用的 fps 就是 60 ，给用户的感觉就是非常流畅，这就是引入 Choreographer 的主要作用

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161605573.png)

当然目前使用 90Hz 刷新率屏幕的手机越来越多，Vsync 周期从 16.6ms 到了 11.1ms，上图中的操作要在更短的时间内完成，对性能的要求也越来越高，具体可以看[新的流畅体验，90Hz 漫谈](https://www.androidperformance.com/2019/05/15/90hz-on-android/) 这篇文章

# Choreographer 简介

Choreographer 扮演 Android 渲染链路中承上启下的角色

1. **承上**：负责接收和处理 App 的各种更新消息和回调，等到 Vsync 到来的时候统一处理。比如集中处理 Input(主要是 Input 事件的处理) 、Animation(动画相关)、Traversal(包括 measure、layout、draw 等操作) ，判断卡顿掉帧情况，记录 CallBack 耗时等
2. **启下**：负责请求和接收 Vsync 信号。接收 Vsync 事件回调(通过 FrameDisplayEventReceiver.onVsync )；请求 Vsync(FrameDisplayEventReceiver.scheduleVsync)

从上面可以看出来， Choreographer 担任的是一个工具人的角色，他之所以重要，是因为通过 **Choreographer + SurfaceFlinger + Vsync + TripleBuffer** 这一套从上到下的机制，保证了 Android App 可以以一个稳定的帧率运行(20fps、90fps 或者 60fps)，减少帧率波动带来的不适感。

了解 Choreographer 还可以帮助 App 开发者知道程序每一帧运行的基本原理，也可以加深对 **Message、Handler、Looper、MessageQueue、Input、Animation、Measure、Layout、Draw** 的理解 , 很多 **APM** 工具也用到了 **Choreographer( 利用 FrameCallback + FrameInfo )** + **MessageQueue ( 利用 IdleHandler )** + **Looper ( 设置自定义 MessageLogging)** 这些组合拳，深入了解了这些之后，再去做优化，脑子里的思路会更清晰。

另外虽然画图是一个比较好的解释流程的好路子，但是我个人不是很喜欢画图，因为平时 Systrace 和 MethodTrace 用的比较多，Systrace 是按从左到右展示整个系统的运行情况的一个工具(包括 cpu、SurfaceFlinger、SystemServer、App 等关键进程)，使用 **Systrace** 和 **MethodTrace** 也可以很方便地展示关键流程。当你对系统代码比较熟悉的时候，看 Systrace 就可以和手机运行的实际情况对应起来。所以下面的文章除了一些网图之外，其他的我会多以 Systrace 来展示。

## 从 Systrace 的角度来看 Choreogrepher 的工作流程[](https://androidperformance.com/2019/10/22/Android-Choreographer/#%E4%BB%8E-Systrace-%E7%9A%84%E8%A7%92%E5%BA%A6%E6%9D%A5%E7%9C%8B-Choreogrepher-%E7%9A%84%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B)

下图以滑动桌面为例子，我们先看一下从左到右滑动桌面的一个完整的预览图（App 进程），可以看到 Systrace 中从左到右，每一个绿色的帧都表示一帧，表示最终我们可以手机上看到的画面

1. 图中每一个灰色的条和白色的条宽度是一个 Vsync 的时间，也就是 16.6ms
2. 每一帧处理的流程：接收到 Vsync 信号回调-> UI Thread –> RenderThread –> SurfaceFlinger(图中未显示)
3. UI Thread 和 RenderThread 就可以完成 App 一帧的渲染，渲染完的 Buffer 抛给 SurfaceFlinger 去合成，然后我们就可以在屏幕上看到这一帧了
4. 可以看到桌面滑动的每一帧耗时都很短（Ui Thread 耗时 + RenderThread 耗时），但是由于 Vsync 的存在，每一帧都会等到 Vsync 才会去做处理

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161605413.png)

有了上面这个整体的概念，我们将 UI Thread 的每一帧放大来看，看看 Choreogrepher 的位置以及 Choreogrepher 是怎么组织每一帧的

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161605195.png)

## Choreographer 的工作流程[](https://androidperformance.com/2019/10/22/Android-Choreographer/#Choreographer-%E7%9A%84%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B)

1. Choreographer 初始化
2. 初始化 FrameHandler ，绑定 Looper
3. 初始化 FrameDisplayEventReceiver ，与 SurfaceFlinger 建立通信用于接收和请求 Vsync
4. 初始化 CallBackQueues
5. SurfaceFlinger 的 appEventThread 唤醒发送 Vsync ，Choreographer 回调 FrameDisplayEventReceiver.onVsync , 进入 Choreographer 的主处理函数 doFrame
6. Choreographer.doFrame 计算掉帧逻辑
7. Choreographer.doFrame 处理 Choreographer 的第一个 callback ： input
8. Choreographer.doFrame 处理 Choreographer 的第二个 callback ： animation
9. Choreographer.doFrame 处理 Choreographer 的第三个 callback ： insets animation
10. Choreographer.doFrame 处理 Choreographer 的第四个 callback ： traversal
11. traversal-draw 中 UIThread 与 RenderThread 同步数据
12. Choreographer.doFrame 处理 Choreographer 的第五个 callback ： commit ?
13. RenderThread 处理绘制命令，将处理好的绘制命令发给 GPU 处理
14. 调用 swapBuffer 提交给 SurfaceFlinger 进行合成（此时 Buffer 并没有真正完成，需要等 CPU 完成后 SurfaceFlinger 才能真正使用，新版本的 Systrace 中有 gpu 的 fence 来标识这个时间）

**第一步初始化完成后，后续就会在步骤 2-9 之间循环**

同时也附上这一帧所对应的 MethodTrace（这里预览一下即可，下面会有详细的大图）

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161606877.png)


下面我们就从源码的角度，来看一下具体的实现

# 源码解析

下面从源码的角度来简单看一下，源码只摘抄了部分重要的逻辑，其他的逻辑则被剔除，另外 Native 部分与 SurfaceFlinger 交互的部分也没有列入，不是本文的重点，有兴趣的可以自己去跟一下。

## Choreographer 的初始化[](https://androidperformance.com/2019/10/22/Android-Choreographer/#Choreographer-%E7%9A%84%E5%88%9D%E5%A7%8B%E5%8C%96)

### Choreographer 的单例初始化

```java
// Thread local storage for the choreographer.  
private static final ThreadLocal<Choreographer> sThreadInstance =  
        new ThreadLocal<Choreographer>() {  
    @Override  
    protected Choreographer initialValue() {  
        // 获取当前线程的 Looper  
        Looper looper = Looper.myLooper();  
        ......  
        // 构造 Choreographer 对象  
        Choreographer choreographer = new Choreographer(looper, VSYNC_SOURCE_APP);  
        if (looper == Looper.getMainLooper()) {  
            mMainInstance = choreographer;  
        }  
        return choreographer;  
    }  
};
```

### Choreographer 的构造函数

```java
private Choreographer(Looper looper, int vsyncSource) {  
    mLooper = looper;  
    // 1. 初始化 FrameHandler  
    mHandler = new FrameHandler(looper);  
    // 2. 初始化 DisplayEventReceiver  
    mDisplayEventReceiver = USE_VSYNC  
            ? new FrameDisplayEventReceiver(looper, vsyncSource)  
            : null;  
    mLastFrameTimeNanos = Long.MIN_VALUE;  
    mFrameIntervalNanos = (long)(1000000000 / getRefreshRate());  
    //3. 初始化 CallbacksQueues  
    mCallbackQueues = new CallbackQueue[CALLBACK_LAST + 1];  
    for (int i = 0; i <= CALLBACK_LAST; i++) {  
        mCallbackQueues[i] = new CallbackQueue();  
    }  
    ......  
}
```

### FrameHandler

```java
private final class FrameHandler extends Handler {  
    ......  
    public void handleMessage(Message msg) {  
        switch (msg.what) {  
            case MSG_DO_FRAME://开始渲染下一帧的操作  
                doFrame(System.nanoTime(), 0);  
                break;  
            case MSG_DO_SCHEDULE_VSYNC://请求 Vsync   
                doScheduleVsync();  
                break;  
            case MSG_DO_SCHEDULE_CALLBACK://处理 Callback  
                doScheduleCallback(msg.arg1);  
                break;  
        }  
    }  
}
```

### Choreographer 初始化链[](https://androidperformance.com/2019/10/22/Android-Choreographer/#Choreographer-%E5%88%9D%E5%A7%8B%E5%8C%96%E9%93%BE)

在 Activity 启动过程，执行完 onResume 后，会调用 Activity.makeVisible()，然后再调用到 addView()， 层层调用会进入如下方法

```java
ActivityThread.handleResumeActivity(IBinder, boolean, boolean, String) (android.app)   
-->WindowManagerImpl.addView(View, LayoutParams) (android.view)   
  -->WindowManagerGlobal.addView(View, LayoutParams, Display, Window) (android.view)   
    -->ViewRootImpl.ViewRootImpl(Context, Display) (android.view)   
    public ViewRootImpl(Context context, Display display) {  
        ......  
        mChoreographer = Choreographer.getInstance();  
        ......  
    }
```

## FrameDisplayEventReceiver 简介[](https://androidperformance.com/2019/10/22/Android-Choreographer/#FrameDisplayEventReceiver-%E7%AE%80%E4%BB%8B)

Vsync 的注册、申请、接收都是通过 FrameDisplayEventReceiver 这个类，所以可以先简单介绍一下。 FrameDisplayEventReceiver 继承 DisplayEventReceiver ， 有三个比较重要的方法

1. onVsync – Vsync 信号回调
2. run – 执行 doFrame
3. scheduleVsync – 请求 Vsync 信号

```java
private final class FrameDisplayEventReceiver extends DisplayEventReceiver implements Runnable {  
    ......  
    @Override  
    public void onVsync(long timestampNanos, long physicalDisplayId, int frame) {  
        ......  
        mTimestampNanos = timestampNanos;  
        mFrame = frame;  
        Message msg = Message.obtain(mHandler, this);  
        msg.setAsynchronous(true);  
        mHandler.sendMessageAtTime(msg, timestampNanos / TimeUtils.NANOS_PER_MS);  
    }  
    @Override  
    public void run() {  
        mHavePendingVsync = false;  
        doFrame(mTimestampNanos, mFrame);  
    }  
      
    public void scheduleVsync() {  
        ......    
        nativeScheduleVsync(mReceiverPtr);  
        ......  
    }  
}
```

## Choreographer 中 Vsync 的注册[](https://androidperformance.com/2019/10/22/Android-Choreographer/#Choreographer-%E4%B8%AD-Vsync-%E7%9A%84%E6%B3%A8%E5%86%8C)

从下面的函数调用栈可以看到，Choreographer 的内部类 FrameDisplayEventReceiver.onVsync 负责接收 Vsync 回调，通知 UIThread 进行数据处理。

那么 FrameDisplayEventReceiver 是通过什么方式在 Vsync 信号到来的时候回调 onVsync 呢？答案是 FrameDisplayEventReceiver 的初始化的时候，最终通过监听文件句柄的形式，其对应的初始化流程如下

android/view/Choreographer.java

```java
private Choreographer(Looper looper, int vsyncSource) {  
    mLooper = looper;  
    mDisplayEventReceiver = USE_VSYNC  
            ? new FrameDisplayEventReceiver(looper, vsyncSource)  
            : null;  
    ......  
}
```

android/view/Choreographer.java

```java
public FrameDisplayEventReceiver(Looper looper, int vsyncSource) {  
    super(looper, vsyncSource);  
}
```

android/view/DisplayEventReceiver.java

```java
public DisplayEventReceiver(Looper looper, int vsyncSource) {  
    ......  
    mMessageQueue = looper.getQueue();  
    mReceiverPtr = nativeInit(new WeakReference<DisplayEventReceiver>(this), mMessageQueue,  
            vsyncSource);  
}
```

nativeInit 后续的代码可以自己跟一下，可以对照这篇文章和源码，由于篇幅比较多，这里就不细写了([https://www.jianshu.com/p/304f56f5d486](https://www.jianshu.com/p/304f56f5d486)) ， 后续梳理好这一块的逻辑后，会在另外的文章更新。

简单来说，FrameDisplayEventReceiver 的初始化过程中，通过 BitTube(本质是一个 socket pair)，来传递和请求 Vsync 事件，当 SurfaceFlinger 收到 Vsync 事件之后，通过 appEventThread 将这个事件通过 BitTube 传给 DisplayEventDispatcher ，DisplayEventDispatcher 通过 BitTube 的接收端监听到 Vsync 事件之后，回调 Choreographer.FrameDisplayEventReceiver.onVsync ，触发开始一帧的绘制，如下图


![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161609680.png)

## Choreographer 处理一帧的逻辑[](https://androidperformance.com/2019/10/22/Android-Choreographer/#Choreographer-%E5%A4%84%E7%90%86%E4%B8%80%E5%B8%A7%E7%9A%84%E9%80%BB%E8%BE%91)

Choreographer 处理绘制的逻辑核心在 Choreographer.doFrame 函数中，从下图可以看到，FrameDisplayEventReceiver.onVsync post 了自己，其 run 方法直接调用了 doFrame 开始一帧的逻辑处理

android/view/Choreographer.java

```java
public void onVsync(long timestampNanos, long physicalDisplayId, int frame) {  
    ......  
    mTimestampNanos = timestampNanos;  
    mFrame = frame;  
    Message msg = Message.obtain(mHandler, this);  
    msg.setAsynchronous(true);  
    mHandler.sendMessageAtTime(msg, timestampNanos / TimeUtils.NANOS_PER_MS);  
}  
public void run() {  
    mHavePendingVsync = false;  
    doFrame(mTimestampNanos, mFrame);  
}
```

doFrame 函数主要做下面几件事

1. 计算掉帧逻辑
2. 记录帧绘制信息
3. 执行 CALLBACK_INPUT、CALLBACK_ANIMATION、CALLBACK_INSETS_ANIMATION、CALLBACK_TRAVERSAL、CALLBACK_COMMIT

### 计算掉帧逻辑

```java
void doFrame(long frameTimeNanos, int frame) {  
    final long startNanos;  
    synchronized (mLock) {  
        ......  
        long intendedFrameTimeNanos = frameTimeNanos;  
        startNanos = System.nanoTime();  
        final long jitterNanos = startNanos - frameTimeNanos;  
        if (jitterNanos >= mFrameIntervalNanos) {  
            final long skippedFrames = jitterNanos / mFrameIntervalNanos;  
            if (skippedFrames >= SKIPPED_FRAME_WARNING_LIMIT) {  
                Log.i(TAG, "Skipped " + skippedFrames + " frames!  "  
                        + "The application may be doing too much work on its main thread.");  
            }  
        }  
        ......  
    }  
    ......  
}
```

Choreographer.doFrame 的掉帧检测比较简单，从下图可以看到，Vsync 信号到来的时候会标记一个 start_time ，执行 doFrame 的时候标记一个 end_time ，这两个时间差就是 Vsync 处理时延，也就是掉帧

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161610180.png)

我们以 Systrace 的掉帧的实际情况来看掉帧的计算逻辑

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161610761.png)

这里需要注意的是，这种方法计算的掉帧，是前一帧的掉帧情况，而不是这一帧的掉帧情况，这个计算方法是有缺陷的，会导致有的掉帧没有被计算到

### 记录帧绘制信息[](https://androidperformance.com/2019/10/22/Android-Choreographer/#%E8%AE%B0%E5%BD%95%E5%B8%A7%E7%BB%98%E5%88%B6%E4%BF%A1%E6%81%AF)

Choreographer 中 FrameInfo 来负责记录帧的绘制信息，doFrame 执行的时候，会把每一个关键节点的绘制时间记录下来，我们使用 dumpsys gfxinfo 就可以看到。当然 Choreographer 只是记录了一部分，剩余的部分在 hwui 那边来记录。

从 FrameInfo 这些标志就可以看出记录的内容，后面我们看 dumpsys gfxinfo 的时候数据就是按照这个来排列的

```java
// Various flags set to provide extra metadata about the current frame  
private static final int FLAGS = 0;  
  
// Is this the first-draw following a window layout?  
public static final long FLAG_WINDOW_LAYOUT_CHANGED = 1;  
  
// A renderer associated with just a Surface, not with a ViewRootImpl instance.  
public static final long FLAG_SURFACE_CANVAS = 1 << 2;  
  
@LongDef(flag = true, value = {  
        FLAG_WINDOW_LAYOUT_CHANGED, FLAG_SURFACE_CANVAS })  
@Retention(RetentionPolicy.SOURCE)  
public @interface FrameInfoFlags {}  
  
// The intended vsync time, unadjusted by jitter  
private static final int INTENDED_VSYNC = 1;  
  
// Jitter-adjusted vsync time, this is what was used as input into the  
// animation & drawing system  
private static final int VSYNC = 2;  
  
// The time of the oldest input event  
private static final int OLDEST_INPUT_EVENT = 3;  
  
// The time of the newest input event  
private static final int NEWEST_INPUT_EVENT = 4;  
  
// When input event handling started  
private static final int HANDLE_INPUT_START = 5;  
  
// When animation evaluations started  
private static final int ANIMATION_START = 6;  
  
// When ViewRootImpl#performTraversals() started  
private static final int PERFORM_TRAVERSALS_START = 7;  
  
// When View:draw() started  
private static final int DRAW_START = 8;
```

doFrame 函数记录从 Vsync time 到 markPerformTraversalsStart 的时间

```java
void doFrame(long frameTimeNanos, int frame) {  
    ......  
    mFrameInfo.setVsync(intendedFrameTimeNanos, frameTimeNanos);  
    // 处理 CALLBACK_INPUT Callbacks   
    mFrameInfo.markInputHandlingStart();  
    // 处理 CALLBACK_ANIMATION Callbacks  
    mFrameInfo.markAnimationsStart();  
    // 处理 CALLBACK_INSETS_ANIMATION Callbacks  
    // 处理 CALLBACK_TRAVERSAL Callbacks  
    mFrameInfo.markPerformTraversalsStart();  
    // 处理 CALLBACK_COMMIT Callbacks  
    ......  
}
```

### 执行 Callbacks

```java
void doFrame(long frameTimeNanos, int frame) {  
    ......  
    // 处理 CALLBACK_INPUT Callbacks   
    doCallbacks(Choreographer.CALLBACK_INPUT, frameTimeNanos);  
    // 处理 CALLBACK_ANIMATION Callbacks  
    doCallbacks(Choreographer.CALLBACK_ANIMATION, frameTimeNanos);  
    // 处理 CALLBACK_INSETS_ANIMATION Callbacks  
    doCallbacks(Choreographer.CALLBACK_INSETS_ANIMATION, frameTimeNanos);  
    // 处理 CALLBACK_TRAVERSAL Callbacks  
    doCallbacks(Choreographer.CALLBACK_TRAVERSAL, frameTimeNanos);  
    // 处理 CALLBACK_COMMIT Callbacks  
    doCallbacks(Choreographer.CALLBACK_COMMIT, frameTimeNanos);  
    ......  
}
```

**Input 回调调用栈**

**input callback** 一般是执行 ViewRootImpl.ConsumeBatchedInputRunnable

android/view/ViewRootImpl.java


```java
final class ConsumeBatchedInputRunnable implements Runnable {  
    @Override  
    public void run() {  
        doConsumeBatchedInput(mChoreographer.getFrameTimeNanos());  
    }  
}  
void doConsumeBatchedInput(long frameTimeNanos) {  
    if (mConsumeBatchedInputScheduled) {  
        mConsumeBatchedInputScheduled = false;  
        if (mInputEventReceiver != null) {  
            if (mInputEventReceiver.consumeBatchedInputEvents(frameTimeNanos)  
                    && frameTimeNanos != -1) {  
                scheduleConsumeBatchedInput();  
            }  
        }  
        doProcessInputEvents();  
    }  
}
```

Input 时间经过处理，最终会传给 DecorView 的 dispatchTouchEvent，这就到了我们熟悉的 Input 事件分发

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161612585.png)

**Animation 回调调用栈**

一般我们接触的多的是调用 View.postOnAnimation 的时候，会使用到 CALLBACK_ANIMATION

```java
public void postOnAnimation(Runnable action) {  
    final AttachInfo attachInfo = mAttachInfo;  
    if (attachInfo != null) {  
        attachInfo.mViewRootImpl.mChoreographer.postCallback(  
                Choreographer.CALLBACK_ANIMATION, action, null);  
    } else {  
        // Postpone the runnable until we know  
        // on which thread it needs to run.  
        getRunQueue().post(action);  
    }  
}
```

那么一般是什么时候回调用到 View.postOnAnimation 呢，我截取了一张图，大家可以自己去看一下，接触最多的应该是 startScroll，Fling 这种操作

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161613925.png)

其调用栈根据其 post 的内容，下面是桌面滑动松手之后的 fling 动画。

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161613975.png)

另外我们的 Choreographer 的 FrameCallback 也是用的 CALLBACK_ANIMATION

```java
public void postFrameCallbackDelayed(FrameCallback callback, long delayMillis) {  
    if (callback == null) {  
        throw new IllegalArgumentException("callback must not be null");  
    }  
  
    postCallbackDelayedInternal(CALLBACK_ANIMATION,  
            callback, FRAME_CALLBACK_TOKEN, delayMillis);  
}
```

**Traversal 调用栈**

```java
void scheduleTraversals() {  
    if (!mTraversalScheduled) {  
        mTraversalScheduled = true;  
        //为了提高优先级，先 postSyncBarrier  
        mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();  
        mChoreographer.postCallback(  
                Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);  
    }  
}  
  
final class TraversalRunnable implements Runnable {  
    @Override  
    public void run() {  
        // 真正开始执行 measure、layout、draw  
        doTraversal();  
    }  
}  
void doTraversal() {  
    if (mTraversalScheduled) {  
        mTraversalScheduled = false;  
        // 这里把 SyncBarrier remove  
mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);  
        // 真正开始  
        performTraversals();  
    }  
}  
private void performTraversals() {  
      // measure 操作  
      if (focusChangedDueToTouchMode || mWidth != host.getMeasuredWidth() || mHeight != host.getMeasuredHeight() || contentInsetsChanged || updatedConfiguration) {  
            performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);  
      }  
      // layout 操作  
      if (didLayout) {  
          performLayout(lp, mWidth, mHeight);  
      }  
      // draw 操作  
      if (!cancelDraw && !newSurface) {  
          performDraw();  
      }  
}
```

**doTraversal 的 TraceView 示例**

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161614455.png)

## 下一帧的 Vsync 请求[](https://androidperformance.com/2019/10/22/Android-Choreographer/#%E4%B8%8B%E4%B8%80%E5%B8%A7%E7%9A%84-Vsync-%E8%AF%B7%E6%B1%82)

由于动画、滑动、Fling 这些操作的存在，我们需要一个连续的、稳定的帧率输出机制。这就涉及到了 Vsync 的请求逻辑，在连续的操作，比如动画、滑动、Fling 这些情况下，每一帧的 doFrame 的时候，都会根据情况触发下一个 Vsync 的申请，这样我们就可以获得连续的 Vsync 信号。

看下面的 scheduleTraversals 调用栈(scheduleTraversals 中会触发 Vsync 请求)

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161614029.png)

我们比较熟悉的 invalidate 和 requestLayout 都会触发 Vsync 信号请求

我们下面以 Animation 为例，看看 Animation 是如何驱动下一个 Vsync ，来持续更新画面的

### ObjectAnimator 动画驱动逻辑[](https://androidperformance.com/2019/10/22/Android-Choreographer/#ObjectAnimator-%E5%8A%A8%E7%94%BB%E9%A9%B1%E5%8A%A8%E9%80%BB%E8%BE%91)

android/animation/ObjectAnimator.java

```java
public void start() {  
    super.start();  
}
```

android/animation/ValueAnimator.java

```java
private void start(boolean playBackwards) {  
    ......  
    addAnimationCallback(0); // 动画 start 的时候添加 Animation Callback   
    ......  
}  
private void addAnimationCallback(long delay) {  
    ......  
    getAnimationHandler().addAnimationFrameCallback(this, delay);  
}
```

android/animation/AnimationHandler.java

```java
public void addAnimationFrameCallback(final AnimationFrameCallback callback, long delay) {  
    if (mAnimationCallbacks.size() == 0) {  
        // post FrameCallback  
        getProvider().postFrameCallback(mFrameCallback);  
    }  
    ......  
}  
  
// 这里的 mFrameCallback 回调 doFrame，里面 post了自己  
private final Choreographer.FrameCallback mFrameCallback = new Choreographer.FrameCallback() {  
    @Override  
    public void doFrame(long frameTimeNanos) {  
        doAnimationFrame(getProvider().getFrameTime());  
        if (mAnimationCallbacks.size() > 0) {  
            // post 自己  
            getProvider().postFrameCallback(this);  
        }  
    }  
};
```

调用 postFrameCallback 会走到 mChoreographer.postFrameCallback ，这里就会触发 Choreographer 的 Vsync 请求逻辑

android/animation/AnimationHandler.java

```java
public void postFrameCallback(Choreographer.FrameCallback callback) {  
    mChoreographer.postFrameCallback(callback);  
}
```

android/view/Choreographer.java

```java
private void postCallbackDelayedInternal(int callbackType,  
        Object action, Object token, long delayMillis) {  
    synchronized (mLock) {  
        final long now = SystemClock.uptimeMillis();  
        final long dueTime = now + delayMillis;  
        mCallbackQueues[callbackType].addCallbackLocked(dueTime, action, token);  
  
        if (dueTime <= now) {  
            // 请求 Vsync scheduleFrameLocked ->scheduleVsyncLocked-> mDisplayEventReceiver.scheduleVsync ->nativeScheduleVsync  
            scheduleFrameLocked(now);  
        } else {  
            Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_CALLBACK, action);  
            msg.arg1 = callbackType;  
            msg.setAsynchronous(true);  
            mHandler.sendMessageAtTime(msg, dueTime);  
        }  
    }  
}
```

通过上面的 Animation.start 设置，利用了 Choreographer.FrameCallback 接口，每一帧都去请求下一个 Vsync  
**动画过程中一帧的 TraceView 示例**

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161616859.png)

## 源码小结[](https://androidperformance.com/2019/10/22/Android-Choreographer/#%E6%BA%90%E7%A0%81%E5%B0%8F%E7%BB%93)

1. **Choreographer** 是线程单例的，而且必须要和一个 Looper 绑定，因为其内部有一个 Handler 需要和 Looper 绑定，一般是 App 主线程的 Looper 绑定
    
2. **DisplayEventReceiver** 是一个 abstract class，其 JNI 的代码部分会创建一个IDisplayEventConnection 的 Vsync 监听者对象。这样，来自 AppEventThread 的 VSYNC 中断信号就可以传递给 Choreographer 对象了。当 Vsync 信号到来时，DisplayEventReceiver 的 onVsync 函数将被调用。
    
3. **DisplayEventReceiver** 还有一个 scheduleVsync 函数。当应用需要绘制UI时，将首先申请一次 Vsync 中断，然后再在中断处理的 onVsync 函数去进行绘制。
    
4. **Choreographer** 定义了一个 **FrameCallback** **interface**，每当 Vsync 到来时，其 doFrame 函数将被调用。这个接口对 Android Animation 的实现起了很大的帮助作用。以前都是自己控制时间，现在终于有了固定的时间中断。
    
5. **Choreographer** 的主要功能是，当收到 Vsync 信号时，去调用使用者通过 postCallback 设置的回调函数。目前一共定义了五种类型的回调，它们分别是：
    
    1. **CALLBACK_INPUT** : 处理输入事件处理有关
    2. **CALLBACK_ANIMATION** ： 处理 Animation 的处理有关
    3. **CALLBACK_INSETS_ANIMATION** ： 处理 Insets Animation 的相关回调
    4. **CALLBACK_TRAVERSAL** : 处理和 UI 等控件绘制有关
    5. **CALLBACK_COMMIT** ： 处理 Commit 相关回调，主要是是用于执行组件 Application/Activity/Service 的 onTrimMemory，在 ApplicationThread 的 scheduleTrimMemory 方法中向 Choreographer 插入的；另外这个 Callback 也提供了一个监测一帧耗时的时机
6. **ListView** 的 Item 初始化(obtain\setup) 会在 input 里面也会在 animation 里面，这取决于
    
7. **CALLBACK_INPUT** 、**CALLBACK_ANIMATION** 会修改 view 的属性，所以要先与 CALLBACK_TRAVERSAL 执行
    

# APM 与 Choreographer

由于 Choreographer 的位置，许多性能监控的手段都是利用 Choreographer 来做的，除了自带的掉帧计算，Choreographer 提供的 FrameCallback 和 FrameInfo 都给 App 暴露了接口，让 App 开发者可以通过这些方法监控自身 App 的性能，其中常用的方法如下：

1. 利用 FrameCallback 的 doFrame 回调
2. 利用 FrameInfo 进行监控
3. 使用 ：adb shell dumpsys gfxinfo framestats
4. 示例 ：adb shell dumpsys gfxinfo com.meizu.flyme.launcher framestats
5. 利用 SurfaceFlinger 进行监控
6. 使用 ：adb shell dumpsys SurfaceFlinger –latency
7. 示例 ：adb shell dumpsys SurfaceFlinger –latency com.meizu.flyme.launcher/com.meizu.flyme.launcher.Launcher#0
8. 利用 SurfaceFlinger PageFlip 机制进行监控
9. 使用 ： adb service call SurfaceFlinger 1013
10. 备注：需要系统权限
11. Choreographer 自身的掉帧计算逻辑
12. BlockCanary 基于 Looper 的性能监控

## 利用 FrameCallback 的 doFrame 回调[](https://androidperformance.com/2019/10/22/Android-Choreographer/#%E5%88%A9%E7%94%A8-FrameCallback-%E7%9A%84-doFrame-%E5%9B%9E%E8%B0%83)

### FrameCallback 接口

```java
public interface FrameCallback {  
    public void doFrame(long frameTimeNanos);  
}
```

### 接口使用

```java
Choreographer.getInstance().postFrameCallback(youOwnFrameCallback );
```

### 接口处理

```java
public void postFrameCallbackDelayed(FrameCallback callback, long delayMillis) {  
    ......  
    postCallbackDelayedInternal(CALLBACK_ANIMATION,  
            callback, FRAME_CALLBACK_TOKEN, delayMillis);  
}
```

TinyDancer 就是使用了这个方法来计算 FPS ([https://github.com/friendlyrobotnyc/TinyDancer](https://github.com/friendlyrobotnyc/TinyDancer))

## 利用 FrameInfo 进行监控[](https://androidperformance.com/2019/10/22/Android-Choreographer/#%E5%88%A9%E7%94%A8-FrameInfo-%E8%BF%9B%E8%A1%8C%E7%9B%91%E6%8E%A7)

adb shell dumpsys gfxinfo framestats

```shell
Window: StatusBar  
Stats since: 17990256398ns  
Total frames rendered: 1562  
Janky frames: 361 (23.11%)  
50th percentile: 6ms  
90th percentile: 23ms  
95th percentile: 36ms  
99th percentile: 101ms  
Number Missed Vsync: 33  
Number High input latency: 683  
Number Slow UI thread: 273  
Number Slow bitmap uploads: 8  
Number Slow issue draw commands: 18  
Number Frame deadline missed: 287  
HISTOGRAM: 5ms=670 6ms=128 7ms=84 8ms=63 9ms=38 10ms=23 11ms=21 12ms=20 13ms=25 14ms=39 15ms=65 16ms=36 17ms=51 18ms=37 19ms=41 20ms=20 21ms=19 22ms=18 23ms=15 24ms=14 25ms=8 26ms=4 27ms=6 28ms=3 29ms=4 30ms=2 31ms=2 32ms=6 34ms=12 36ms=10 38ms=9 40ms=3 42ms=4 44ms=5 46ms=8 48ms=6 53ms=6 57ms=4 61ms=1 65ms=0 69ms=2 73ms=2 77ms=3 81ms=4 85ms=1 89ms=2 93ms=0 97ms=2 101ms=1 105ms=1 109ms=1 113ms=1 117ms=1 121ms=2 125ms=1 129ms=0 133ms=1 150ms=2 200ms=3 250ms=0 300ms=1 350ms=1 400ms=0 450ms=0 500ms=0 550ms=0 600ms=0 650ms=0   
  
---PROFILEDATA---  
Flags,IntendedVsync,Vsync,OldestInputEvent,NewestInputEvent,HandleInputStart,AnimationStart,PerformTraversalsStart,DrawStart,SyncQueued,SyncStart,IssueDrawCommandsStart,SwapBuffers,FrameCompleted,DequeueBufferDuration,QueueBufferDuration,  
0,10158314881426,10158314881426,9223372036854775807,0,10158315693363,10158315760759,10158315769821,10158316032165,10158316627842,10158316838988,10158318055915,10158320387269,10158321770654,428000,773000,  
0,10158332036261,10158332036261,9223372036854775807,0,10158332799196,10158332868519,10158332877269,10158333137738,10158333780654,10158333993206,10158335078467,10158337689561,10158339307061,474000,885000,  
0,10158348665353,10158348665353,9223372036854775807,0,10158349710238,10158349773102,10158349780863,10158350405863,10158351135967,10158351360446,10158352300863,10158354305654,10158355814509,471000,836000,  
0,10158365296729,10158365296729,9223372036854775807,0,10158365782373,10158365821019,10158365825238,10158365975290,10158366547946,10158366687217,10158367240706,10158368429248,10158369291852,269000,476000,
```

## 利用 SurfaceFlinger 进行监控[](https://androidperformance.com/2019/10/22/Android-Choreographer/#%E5%88%A9%E7%94%A8-SurfaceFlinger-%E8%BF%9B%E8%A1%8C%E7%9B%91%E6%8E%A7)

命令解释：

1. 数据的单位是纳秒，时间是以开机时间为起始点
2. 每一次的命令都会得到128行的帧相关的数据

数据：

1. 第一行数据，表示刷新的时间间隔refresh_period
2. 第1列：这一部分的数据表示应用程序绘制图像的时间点
3. 第2列：在SF(软件)将帧提交给H/W(硬件)绘制之前的垂直同步时间，也就是每帧绘制完提交到硬件的时间戳，该列就是垂直同步的时间戳
4. 第3列：在SF将帧提交给H/W的时间点，算是H/W接受完SF发来数据的时间点，绘制完成的时间点。

**掉帧 jank 计算**

每一行都可以通过下面的公式得到一个值，该值是一个标准，我们称为jankflag，如果当前行的jankflag与上一行的jankflag发生改变，那么就叫掉帧

ceil((C - A) / refresh-period)

## 利用 SurfaceFlinger PageFlip 机制进行监控

```java
Parcel data = Parcel.obtain();  
Parcel reply = Parcel.obtain();  
                data.writeInterfaceToken("android.ui.ISurfaceComposer");  
mFlinger.transact(1013, data, reply, 0);  
final int pageFlipCount = reply.readInt();  
  
final long now = System.nanoTime();  
final int frames = pageFlipCount - mLastPageFlipCount;  
final long duration = now - mLastUpdateTime;  
mFps = (float) (frames * 1e9 / duration);  
mLastPageFlipCount = pageFlipCount;  
mLastUpdateTime = now;  
reply.recycle();  
data.recycle();
```

## Choreographer 自身的掉帧计算逻辑[](https://androidperformance.com/2019/10/22/Android-Choreographer/#Choreographer-%E8%87%AA%E8%BA%AB%E7%9A%84%E6%8E%89%E5%B8%A7%E8%AE%A1%E7%AE%97%E9%80%BB%E8%BE%91)

SKIPPED_FRAME_WARNING_LIMIT 默认为30 , 由 debug.choreographer.skipwarning 这个属性控制

```java
if (jitterNanos >= mFrameIntervalNanos) {  
    final long skippedFrames = jitterNanos / mFrameIntervalNanos;  
    if (skippedFrames >= SKIPPED_FRAME_WARNING_LIMIT) {  
        Log.i(TAG, "Skipped " + skippedFrames + " frames!  "  
                + "The application may be doing too much work on its main thread.");  
    }  
}
```

## BlockCanary[](https://androidperformance.com/2019/10/22/Android-Choreographer/#BlockCanary)

Blockcanary 做性能监控使用的是 Looper 的消息机制，通过对 MessageQueue 中每一个 Message 的前后进行记录，打到监控性能的目的

android/os/Looper.java

```java
public static void loop() {  
    ...  
    for (;;) {  
        ...  
        // This must be in a local variable, in case a UI event sets the logger  
        Printer logging = me.mLogging;  
        if (logging != null) {  
            logging.println(">>>>> Dispatching to " + msg.target + " " +  
                    msg.callback + ": " + msg.what);  
        }  
        msg.target.dispatchMessage(msg);  
        if (logging != null) {  
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);  
        }  
        ...  
    }  
}
```

# MessageQueue 与 Choreographer

所谓的异步消息其实就是这样的，我们可以通过 enqueueBarrier 往消息队列中插入一个 Barrier，那么队列中执行时间在这个 Barrier 以后的同步消息都会被这个 Barrier 拦截住无法执行，直到我们调用 removeBarrier 移除了这个 Barrier，而异步消息则没有影响，消息默认就是同步消息，除非我们调用了 Message 的 setAsynchronous，这个方法是隐藏的。只有在初始化 Handler 时通过参数指定往这个 Handler 发送的消息都是异步的，这样在 Handler 的 enqueueMessage 中就会调用 Message 的 setAsynchronous 设置消息是异步的，从上面 Handler.enqueueMessage 的代码中可以看到。

所谓异步消息，其实只有一个作用，就是在设置 Barrier 时仍可以不受 Barrier 的影响被正常处理，如果没有设置 Barrier，异步消息就与同步消息没有区别，可以通过 removeSyncBarrier 移除 Barrier

## SyncBarrier 在 Choreographer 中使用的一个示例[](https://androidperformance.com/2019/10/22/Android-Choreographer/#SyncBarrier-%E5%9C%A8-Choreographer-%E4%B8%AD%E4%BD%BF%E7%94%A8%E7%9A%84%E4%B8%80%E4%B8%AA%E7%A4%BA%E4%BE%8B)

scheduleTraversals 的时候 postSyncBarrier

```java
void scheduleTraversals() {  
    if (!mTraversalScheduled) {  
        mTraversalScheduled = true;  
        //为了提高优先级，先 postSyncBarrier  
        mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();  
        mChoreographer.postCallback(  
                Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);  
    }  
}
```

doTraversal 的时候 removeSyncBarrier

```java
void doTraversal() {  
    if (mTraversalScheduled) {  
        mTraversalScheduled = false;  
        // 这里把 SyncBarrier remove  
mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);  
        // 真正开始  
        performTraversals();  
    }  
}
```

Choreographer post Message 的时候，会把这些消息设为 Asynchronous ，这样 Choreographer 中的这些 Message 的优先级就会比较高，

```java
Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_CALLBACK, action);  
msg.arg1 = callbackType;  
msg.setAsynchronous(true);  
mHandler.sendMessageAtTime(msg, dueTime);
```

# 厂商优化

系统厂商由于可以直接修改源码，也利用这方面的便利，做一些功能和优化，不过由于保密的问题，代码就不直接放上来了，我可以大概说一下思路，感兴趣的可以私下讨论

## 移动事件优化[](https://androidperformance.com/2019/10/22/Android-Choreographer/#%E7%A7%BB%E5%8A%A8%E4%BA%8B%E4%BB%B6%E4%BC%98%E5%8C%96)

Choreographer 本身是没有 input 消息的， 不过修改源码之后，input 消息可以直接给到 Choreographer 这里， 有了这些 Input 消息，Choreographer 就可以做一些事情，比如说提前响应，不去等 Vsync

## 后台动画优化[](https://androidperformance.com/2019/10/22/Android-Choreographer/#%E5%90%8E%E5%8F%B0%E5%8A%A8%E7%94%BB%E4%BC%98%E5%8C%96)

当一个 Android App 退到后台之后，只要他没有被杀死，那么他做什么事情大家都不要奇怪，因为这就是 Android。有的 App 退到后台之后还在持续调用 Choreographer 中的 Animation Callback，而这个 Callback 的执行完全是无意义的，而且用户还不知道，但是对 cpu 的占用是比较高的。

所以在 Choreographer 中会针对这种情况做优化，禁止不符合条件的 App 在后台继续无用的操作

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161623313.png)


## 帧绘制优化[](https://androidperformance.com/2019/10/22/Android-Choreographer/#%E5%B8%A7%E7%BB%98%E5%88%B6%E4%BC%98%E5%8C%96)

和移动事件优化一样，由于有了 Input 事件的信息，在某些场景下我们可以通知 SurfaceFlinger 不用去等待 Vsync 直接做合成操作

## 应用启动优化[](https://androidperformance.com/2019/10/22/Android-Choreographer/#%E5%BA%94%E7%94%A8%E5%90%AF%E5%8A%A8%E4%BC%98%E5%8C%96)

我们前面说，主线程的所有操作都是给予 Message 的 ，如果某个操作，非重要的 Message 被排列到了队列后面，那么对这个操作产生影响；而通过重新排列 MessageQueue，在应用启动的时候，把启动相关的重要的启动 Message 放到队列前面，来起到加快启动速度的作用

## 高帧率优化[](https://androidperformance.com/2019/10/22/Android-Choreographer/#%E9%AB%98%E5%B8%A7%E7%8E%87%E4%BC%98%E5%8C%96)

90/120 fps 的手机上 ， Vsync 间隔从 16.6ms 变成了 11.1ms/8.8ms ，这带来了巨大的性能和功耗挑战，如何在一帧内完成渲染的必要操作，是手机厂商必须要思考和优化的地方：

1. 超级 App 的性能表现以及优化
2. 游戏高帧率合作
3. 120fps、90 fps 和 60 fps 相互切换的逻辑

# 参考资料

1. [https://www.jianshu.com/p/304f56f5d486](https://www.jianshu.com/p/304f56f5d486)
2. [http://gityuan.com/2017/02/25/choreographer/](http://gityuan.com/2017/02/25/choreographer/)
3. [https://developer.android.com/reference/android/view/Choreographer](https://developer.android.com/reference/android/view/Choreographer)
4. [https://www.jishuwen.com/d/2Vcc](https://www.jishuwen.com/d/2Vcc)
5. [https://juejin.im/entry/5c8772eee51d456cda2e8099](https://juejin.im/entry/5c8772eee51d456cda2e8099)
6. [Android 开发高手课](https://time.geekbang.org/column/intro/142)

# 原文链接

1. [Systrace 基础知识 - Vsync-App ：基于 Choreographer 的渲染机制详解](https://androidperformance.com/2019/10/22/Android-Choreographer/)
