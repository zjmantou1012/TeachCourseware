---
author: zjmantou
title: HandlerThread
time: 2023-10-31 周二
tags:
  - 技术
  - Android
---
在Android源码中，`HandlerThread`是一个用于创建后台线程和处理消息/任务的类。它在Android系统中的许多地方都被使用到，但是没有一个特定的地方是直接使用`HandlerThread`的例子。相反，`HandlerThread`通常在应用程序代码中使用，以创建后台线程并在其中执行任务。

以下是在Android源码中一些使用`HandlerThread`的示例：

1. 在`ActivityThread`中，`HandlerThread`被用于创建主线程（UI线程）。在`ActivityThread.main()`方法中，`ActivityThread$H`实例被创建，它实际上是一个`HandlerThread`的实例。这个实例用于处理与UI线程相关的消息和任务。
2. 在`AsyncTaskLoader`中，`HandlerThread`被用于创建后台线程，以在异步加载数据时执行任务。`AsyncTaskLoader`类的内部类`WorkerRunnable`实现了`Runnable`接口，并在后台线程中执行加载任务。它使用`HandlerThread`来创建和启动后台线程。
3. 在一些应用程序代码中，例如在`DownloadManager`中，`HandlerThread`被用于创建后台线程来处理文件下载任务。在下载过程中，`HandlerThread`用于处理下载任务的消息和回调。

需要注意的是，这些只是使用`HandlerThread`的一些示例，并不是所有的代码中都直接使用了`HandlerThread`。实际上，Android框架和应用程序代码中使用了许多不同的机制来处理线程和任务，其中许多机制可能间接地使用了`HandlerThread`。

## ServiceThread

ServiceThread 继承自 HandlerThread ，下面介绍的几个工作线程都是继承自 ServiceThread ，分别实现不同的功能，根据线程功能不同，其线程优先级也不同：UIThread、IoThread、DisplayThread、AnimationThread、FgThread、SurfaceAnimationThread

每个 Thread 都有自己的 Looper 、Thread 和 MessageQueue，互相不会影响。Android 系统根据功能，会使用不同的 Thread 来完成。