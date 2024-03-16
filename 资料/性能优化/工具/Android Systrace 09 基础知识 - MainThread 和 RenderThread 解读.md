---
author: zjmantou
title: Android Systrace 09 基础知识 - MainThread 和 RenderThread 解读
time: 2024-03-16 周六
tags:
  - Android
  - 资料
  - 性能优化
  - Systrace
  - Tools
---
本文是 Systrace 系列文章的第九篇，主要是是介绍 Android App 中的 MainThread 和 RenderThread，也就是大家熟悉的**主线程**和**渲染线程**。文章会从 Systrace 的角度来看 MainThread 和 RenderThread 的工作流程，以及涉及到的相关知识：卡顿、软件渲染、掉帧计算等

本系列的目的是通过 Systrace 这个工具，从另外一个角度来看待 Android 系统整体的运行，同时也从另外一个角度来对 Framework 进行学习。也许你看了很多讲 Framework 的文章，但是总是记不住代码，或者不清楚其运行的流程，也许从 Systrace 这个图形化的角度，你可以理解的更深入一些

# 正文

这里以滑动列表为例 ，我们截取主线程和渲染线程**一帧**的工作流程(每一帧都会遵循这个流程，不过有的帧需要处理的事情多，有的帧需要处理的事情少) ，重点看 “UI Thread ” 和 RenderThread 这两行

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161625956.png)


**这张图对应的工作流程如下**

1. 主线程处于 Sleep 状态，等待 Vsync 信号
2. Vsync 信号到来，主线程被唤醒，Choreographer 回调 FrameDisplayEventReceiver.onVsync 开始一帧的绘制
3. 处理 App 这一帧的 Input 事件(如果有的话)
4. 处理 App 这一帧的 Animation 事件(如果有的话)
5. 处理 App 这一帧的 Traversal 事件(如果有的话)
6. 主线程与渲染线程同步渲染数据，同步结束后，主线程结束一帧的绘制，可以继续处理下一个 Message(如果有的话，IdleHandler 如果不为空，这时候也会触发处理)，或者进入 Sleep 状态等待下一个 Vsync
7. 渲染线程首先需要从 BufferQueue 里面取一个 Buffer(dequeueBuffer) , 进行数据处理之后，调用 OpenGL 相关的函数，真正地进行渲染操作，然后将这个渲染好的 Buffer 还给 BufferQueue (queueBuffer) , SurfaceFlinger 在 Vsync-SF 到了之后，将所有准备好的 Buffer 取出进行合成(这个流程在讲 SurfaceFlinger 的时候会提到)

上面这个流程在 [Android 基于 Choreographer 的渲染机制详解](https://www.androidperformance.com/2019/10/22/Android-Choreographer/) 这篇文章里面已经介绍的很详细了，包括每一帧的 doFrame 都在做什么、卡顿计算的原理、APM 相关. 没有看过这篇文章的同学，建议先去扫一眼

那么这篇文章我们主要从 [Android 基于 Choreographer 的渲染机制详解](https://www.androidperformance.com/2019/10/22/Android-Choreographer/) 这篇文章没有讲到的几个点来入手，帮你更好地理解主线程和渲染线程

1. 主线程的发展
2. 主线程的创建
3. 渲染线程的创建
4. 主线程和渲染线程的分工
5. 游戏的主线程与渲染线程
6. Flutter 的主线程和渲染线程

# 主线程的创建

Android App 的进程是基于 Linux 的，其管理也是基于 Linux 的进程管理机制，所以其创建也是调用了 fork 函数

frameworks/base/core/jni/com_android_internal_os_Zygote.cpp

`pid_t pid = fork();`

Fork 出来的进程，我们这里可以把他看做主线程，但是这个线程还没有和 Android 进行连接，所以无法处理 Android App 的 Message ；由于 Android App 线程运行**基于消息机制** ，那么这个 Fork 出来的主线程需要和 Android 的 Message 消息绑定，才能处理 Android App 的各种 Message

这里就引入了 **ActivityThread** ，确切的说，ActivityThread 应该起名叫 ProcessThread 更贴切一些。ActivityThread 连接了 Fork 出来的进程和 App 的 Message ，他们的通力配合组成了我们熟知的 Android App 主线程。所以说 ActivityThread 其实并不是一个 Thread，而是他初始化了 Message 机制所需要的 MessageQueue、Looper、Handler ，而且其 Handler 负责处理大部分 Message 消息，所以我们习惯上觉得 ActivityThread 是主线程，其实他只是主线程的一个逻辑处理单元。

## ActivityThread 的创建[](https://www.androidperformance.com/2019/11/06/Android-Systrace-MainThread-And-RenderThread/#ActivityThread-%E7%9A%84%E5%88%9B%E5%BB%BA)

App 进程 fork 出来之后，回到 App 进程，查找 ActivityThread 的 Main函数

com/android/internal/os/ZygoteInit.java

```java
static final Runnable childZygoteInit(  
        int targetSdkVersion, String[] argv, ClassLoader classLoader) {  
    RuntimeInit.Arguments args = new RuntimeInit.Arguments(argv);  
    return RuntimeInit.findStaticMain(args.startClass, args.startArgs, classLoader);  
}
```

这里的 startClass 就是 ActivityThread，找到之后调用，逻辑就到了 ActivityThread的main函数

android/app/ActivityThread.java

```java
public static void main(String[] args) {  
    //1. 初始化 Looper、MessageQueue  
    Looper.prepareMainLooper();  
    // 2. 初始化 ActivityThread  
    ActivityThread thread = new ActivityThread();  
    // 3. 主要是调用 AMS.attachApplicationLocked，同步进程信息，做一些初始化工作  
    thread.attach(false, startSeq);  
    // 4. 获取主线程的 Handler，这里是 H ，基本上 App 的 Message 都会在这个 Handler 里面进行处理   
    if (sMainThreadHandler == null) {  
        sMainThreadHandler = thread.getHandler();  
    }  
    // 5. 初始化完成，Looper 开始工作  
    Looper.loop();  
}
```

注释里面都很清楚，这里就不详细说了，main 函数处理完成之后，主线程就算是正式上线开始工作，其 Systrace 流程如下：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161626702.png)

## ActivityThread 的功能[](https://www.androidperformance.com/2019/11/06/Android-Systrace-MainThread-And-RenderThread/#ActivityThread-%E7%9A%84%E5%8A%9F%E8%83%BD)

另外我们经常说的，Android 四大组件都是运行在主线程上的，其实这里也很好理解，看一下 ActivityThread 的 Handler 的 Message 就知道了

```java
class H extends Handler { //摘抄了部分  
    public static final int BIND_APPLICATION        = 110;  
    public static final int EXIT_APPLICATION        = 111;  
    public static final int RECEIVER                = 113;  
    public static final int CREATE_SERVICE          = 114;  
    public static final int STOP_SERVICE            = 116;  
    public static final int BIND_SERVICE            = 121;  
    public static final int UNBIND_SERVICE          = 122;  
    public static final int DUMP_SERVICE            = 123;  
    public static final int REMOVE_PROVIDER         = 131;  
    public static final int DISPATCH_PACKAGE_BROADCAST = 133;  
    public static final int DUMP_PROVIDER           = 141;  
    public static final int UNSTABLE_PROVIDER_DIED  = 142;  
    public static final int INSTALL_PROVIDER        = 145;  
    public static final int ON_NEW_ACTIVITY_OPTIONS = 146;  
}
```

可以看到，进程创建、Activity 启动、Service 的管理、Receiver 的管理、Provider 的管理这些都会在这里处理，然后进到具体的 handleXXX

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161627394.png)

# 渲染线程的创建和发展

主线程讲完了我们来讲渲染线程，渲染线程也就是 RenderThread ，最初的 Android 版本里面是没有渲染线程的，渲染工作都是在主线程完成，使用的也都是 CPU ，调用的是 libSkia 这个库，RenderThread 是在 Android Lollipop 中新加入的组件，负责承担一部分之前主线程的渲染工作，减轻主线程的负担

## 软件绘制[](https://www.androidperformance.com/2019/11/06/Android-Systrace-MainThread-And-RenderThread/#%E8%BD%AF%E4%BB%B6%E7%BB%98%E5%88%B6)

我们一般提到的硬件加速，指的就是 GPU 加速，这里可以理解为用 RenderThread 调用 GPU 来进行渲染加速 。 硬件加速在目前的 Android 中是默认开启的， 所以如果我们什么都不设置，那么我们的进程默认都会有主线程和渲染线程(有可见的内容)。我们如果在 App 的 AndroidManifest 里面，在 Application 标签里面加一个

`android:hardwareAccelerated="false"`

我们就可以关闭硬件加速，系统检测到你这个 App 关闭了硬件加速，就不会初始化 RenderThread ，直接 cpu 调用 libSkia 来进行渲染。其 Systrace 的表现如下

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161627606.png)

与这篇文章开头的开了硬件加速的那个图对比，可以看到主线程由于要进行渲染工作，所以执行的时间变长了，也更容易出现卡顿，同时帧与帧直接的空闲间隔也变短了，使得其他 Message 的执行时间被压缩

## 硬件加速绘制[](https://www.androidperformance.com/2019/11/06/Android-Systrace-MainThread-And-RenderThread/#%E7%A1%AC%E4%BB%B6%E5%8A%A0%E9%80%9F%E7%BB%98%E5%88%B6)

正常情况下，硬件加速是开启的，主线程的 draw 函数并没有真正的执行 drawCall ，而是把要 draw 的内容记录到 DIsplayList 里面，同步到 RenderThread 中，一旦同步完成，主线程就可以被释放出来做其他的事情，RenderThread 则继续进行渲染工作

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161628313.png)


## 渲染线程初始化[](https://www.androidperformance.com/2019/11/06/Android-Systrace-MainThread-And-RenderThread/#%E6%B8%B2%E6%9F%93%E7%BA%BF%E7%A8%8B%E5%88%9D%E5%A7%8B%E5%8C%96)

渲染线程初始化在真正需要 draw 内容的时候，一般我们启动一个 Activity ，在第一个 draw 执行的时候，会去检测渲染线程是否初始化，如果没有则去进行初始化

android/view/ViewRootImpl.java

```java
mAttachInfo.mThreadedRenderer.initializeIfNeeded(  
        mWidth, mHeight, mAttachInfo, mSurface, surfaceInsets);
```

后续直接调用 draw

android/graphics/HardwareRenderer.java

```java
mAttachInfo.mThreadedRenderer.draw(mView, mAttachInfo, this);  
void draw(View view, AttachInfo attachInfo, DrawCallbacks callbacks) {  
    final Choreographer choreographer = attachInfo.mViewRootImpl.mChoreographer;  
    choreographer.mFrameInfo.markDrawStart();  
  
    updateRootDisplayList(view, callbacks);  
  
    if (attachInfo.mPendingAnimatingRenderNodes != null) {  
        final int count = attachInfo.mPendingAnimatingRenderNodes.size();  
        for (int i = 0; i < count; i++) {  
            registerAnimatingRenderNode(  
                    attachInfo.mPendingAnimatingRenderNodes.get(i));  
        }  
        attachInfo.mPendingAnimatingRenderNodes.clear();  
        attachInfo.mPendingAnimatingRenderNodes = null;  
    }  
  
    int syncResult = syncAndDrawFrame(choreographer.mFrameInfo);  
    if ((syncResult & SYNC_LOST_SURFACE_REWARD_IF_FOUND) != 0) {  
        setEnabled(false);  
        attachInfo.mViewRootImpl.mSurface.release();  
        attachInfo.mViewRootImpl.invalidate();  
    }  
    if ((syncResult & SYNC_REDRAW_REQUESTED) != 0) {  
        attachInfo.mViewRootImpl.invalidate();  
    }  
}
```

上面的 draw 只是更新 DIsplayList ，更新结束后，调用 syncAndDrawFrame ，通知渲染线程开始工作，主线程释放。渲染线程的核心实现在 libhwui 库里面，其代码位于 frameworks/base/libs/hwui

frameworks/base/libs/hwui/renderthread/RenderProxy.cpp

```c++
int RenderProxy::syncAndDrawFrame() {  
    return mDrawFrameTask.drawFrame();  
}
```

关于 RenderThread 的工作流程这里就不细说了，后续会有专门的篇幅来讲解这个，目前 hwui 这一块的流程也有很多优秀的文章，大家可以对照文章和源码来看，其核心流程在 Systrace 上的表现如下:

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161633682.png)

## 主线程和渲染线程的分工[](https://www.androidperformance.com/2019/11/06/Android-Systrace-MainThread-And-RenderThread/#%E4%B8%BB%E7%BA%BF%E7%A8%8B%E5%92%8C%E6%B8%B2%E6%9F%93%E7%BA%BF%E7%A8%8B%E7%9A%84%E5%88%86%E5%B7%A5)

主线程负责处理进程 Message、处理 Input 事件、处理 Animation 逻辑、处理 Measure、Layout、Draw ，更新 DIsplayList ，但是不涉及 SurfaceFlinger 打交道；渲染线程负责渲染渲染相关的工作，一部分工作也是 CPU 来完成的，一部分操作是调用 OpenGL 函数来完成的

当启动硬件加速后，在 Measure、Layout、Draw 的 Draw 这个环节，Android 使用 DisplayList 进行绘制而非直接使用 CPU 绘制每一帧。DisplayList 是一系列绘制操作的记录，抽象为 RenderNode 类，这样间接的进行绘制操作的优点如下

1. DisplayList 可以按需多次绘制而无须同业务逻辑交互
2. 特定的绘制操作（如 translation， scale 等）可以作用于整个 DisplayList 而无须重新分发绘制操作
3. 当知晓了所有绘制操作后，可以针对其进行优化：例如，所有的文本可以一起进行绘制一次
4. 可以将对 DisplayList 的处理转移至另一个线程（也就是 RenderThread）
5. 主线程在 sync 结束后可以处理其他的 Message，而不用等待 RenderThread 结束

RenderThread 的具体流程大家可以看这篇文章 ： [http://www.cocoachina.com/articles/35302](http://www.cocoachina.com/articles/35302)

# 游戏的主线程与渲染线程

游戏大多使用单独的渲染线程，有单独的 Surface ，直接跟 SurfaceFlinger 进行交互，其主线程的存在感比较低，绝大部分的逻辑都是自己在自己的渲染线程里面实现的。

大家可以看一下王者荣耀对应的 Systrace ，重点看应用进程和 SurfaceFlinger 进程（30fps）

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161633945.png)

可以看到王者荣耀主线程的主要工作，就是把 Input 事件传给 Unity 的渲染线程，渲染线程收到 Input 事件之后，进行逻辑处理，画面更新等。

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161634380.png)

# Flutter 的主线程和渲染线程

这里提一下 Flutter App 在 Systrace 上的表现，由于 Flutter 的渲染是基于 libSkia 的，所以它也没有 RenderThread ，而是他自建的 RenderEngine ， Flutter 比较重要的两个线程是 ui 线程和 gpu 线程，对应到下面提到的  Framework 和 Engine 两层

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161634586.png)

Flutter 中也会监听 Vsync 信号 ，其 VsyncView 中会以 postFrameCallback 的形式，监听 doFrame 回调，然后调用 nativeOnVsync ，将 Vsync 到来的信息传给 Flutter UI 线程，开始一帧的绘制。

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161634401.png)

可以看到 Flutter 的思路跟游戏开发的思路差不多，不依赖具体的平台，自建渲染管道，更新快，跨平台优势明显。

Flutter SDK 自带 Skia 库，不用等系统升级就可以用到最新的 Skia 库，而且 Google 团队在 Skia 上做了很多优化，所以官方号称性能可以媲美原生应用

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161634455.png)

Flutter 的框架分为 Framework 和 Engine 两层，应用是基于 Framework 层开发的，Framework 负责渲染中的 Build，Layout，Paint，生成 Layer 等环节。Engine 层是 C++实现的渲染引擎，负责把 Framework 生成的 Layer 组合，生成纹理，然后通过 Open GL 接口向 GPU 提交渲染数据。

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161635720.png)

当需要更新 UI 的时候，Framework 通知 Engine，Engine 会等到下个 Vsync 信号到达的时候，会通知 Framework，然后 Framework 会进行 animations, build，layout，compositing，paint，最后生成 layer 提交给 Engine。Engine 会把 layer 进行组合，生成纹理，最后通过 Open Gl 接口提交数据给 GPU，GPU 经过处理后在显示器上面显示。整个流程如下图：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161635089.png)


# 性能

如果主线程需要处理所有任务，则执行耗时较长的操作（例如，网络访问或数据库查询）将会阻塞整个界面线程。一旦被阻塞，线程将无法分派任何事件，包括绘图事件。主线程执行超时通常会带来两个问题

1. 卡顿：如果主线程 + 渲染线程每一帧的执行都超过 16.6ms(60fps 的情况下)，那么就可能会出现掉帧。
2. 卡死：如果界面线程被阻塞超过几秒钟时间（根据组件不同 , 这里的阈值也不同），用户会看到 “[应用无响应](http://developer.android.google.cn/guide/practices/responsiveness.html)” (ANR) 对话框(部分厂商屏蔽了这个弹框,会直接 Crash 到桌面)

对于用户来说，这两个情况都是用户不愿意看到的，所以对于 App 开发者来说，两个问题是发版本之前必须要解决的，ANR 这个由于有详细的调用栈，所以相对来说比较好定位；但是间歇性卡顿这个，可能就需要使用工具来进行分析了：Systrace + TraceView，所以理解主线程和渲染线程的关系和他们的工作原理是非常重要的，这也是本系列的一个初衷

另外关于卡顿，可以参考下面三篇文章，你的 App 卡顿不一定是你 App 的问题，也有可能是系统的问题，不过不管怎么说，首先要会分析卡顿问题。

1. [Android 中的卡顿丢帧原因概述 - 方法论](https://www.androidperformance.com/2019/09/05/Android-Jank-Debug/)
2. [Android 中的卡顿丢帧原因概述 - 系统篇](https://www.androidperformance.com/2019/09/05/Android-Jank-Due-To-System/)
3. [Android 中的卡顿丢帧原因概述 - 应用篇](https://www.androidperformance.com/2019/09/05/Android-Jank-Due-To-App/)

# 参考

1. [https://juejin.im/post/5a9e01c3f265da239d48ce32](https://juejin.im/post/5a9e01c3f265da239d48ce32)
2. [http://www.cocoachina.com/articles/35302](http://www.cocoachina.com/articles/35302)
3. [https://juejin.im/post/5b7767fef265da43803bdc65](https://juejin.im/post/5b7767fef265da43803bdc65)
4. [http://gityuan.com/2019/06/15/flutter_ui_draw/](http://gityuan.com/2019/06/15/flutter_ui_draw/)
5. [https://developer.android.google.cn/guide/components/processes-and-threads](https://developer.android.google.cn/guide/components/processes-and-threads)

# 附件

本文涉及到的附件也上传了，各位下载后解压，使用 **Chrome** 浏览器打开即可

[点此链接下载文章所涉及到的 Systrace 附件](https://github.com/Gracker/SystraceForBlog/tree/master/Android_Systrace-MainThread-RenderThread)

# 本文其他地址

由于博客留言交流不方便，点赞或者交流，可以移步本文的知乎或者掘金页面  
[掘金 - Systrace 基础知识 - MainThread 和 RenderThread 解读](https://juejin.im/post/5dc68556f265da4cff702742)

# 原文链接

1. [Systrace 基础知识 - MainThread 和 RenderThread 解读](https://www.androidperformance.com/2019/11/06/Android-Systrace-MainThread-And-RenderThread/)