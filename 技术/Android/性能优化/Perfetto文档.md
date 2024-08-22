---
author: zjmantou
title: Perfetto文档
time: 2024-08-22 周四
tags:
  - 性能优化
  - Perfetto
---
# end_state

| end_state 结束状态 | Translation 翻译               |
| -------------- | ---------------------------- |
| R              | Runnable 可运行                 |
| R+ 右+          | Runnable (Preempted) 可运行（抢占） |
| S              | Sleeping 睡眠                  |
| D              | Uninterruptible Sleep 不间断睡眠  |
| T              | Stopped 已停止                  |
| t              | Traced 追踪                    |
| X              | Exit (Dead) 退出（死）            |
| Z              | Exit (Zombie) 退出（僵尸）         |
| x              | Task Dead 任务死亡               |
| I              | Idle 闲置的                     |
| K              | Wake Kill 唤醒杀                |
| W              | Waking 醒来                    |
| P              | Parked 停放                    |
| N              | No Load 无负载                  |

# 系统调用 

以`sys_`开头 

sys_epoll_pwait 、sys_recvfrom 、sys_ioctl、sys_recvfrom、sys_getuid 

# timeline 

https://perfetto.dev/docs/data-sources/frametimeline 



![](https://perfetto.dev/docs/images/frametimeline/timeline_tracks.png) 

## Expected Timeline 

预期时间线 每个切片代表给予应用程序渲染帧的时间。为了避免系统卡顿，应用程序预计会在此时间范围内完成。开始时间是 Choreographer 回调计划运行的时间。 

## Actual Timeline 

实际时间线 这些切片表示应用程序完成帧（包括 GPU 工作）并将其发送到 SurfaceFlinger 进行合成所花费的实际时间。开始时间是Choreographer#doFrame或AChoreographer_vsyncCallback开始运行的时间。这里切片的结束时间表示max(gpu time, post time) 。发布时间是应用程序的框架发布到 SurfaceFlinger 的时间 

![](https://perfetto.dev/docs/images/frametimeline/selection.png)

### Present Type 

帧是提前、准时还是延迟 

### On time finish 

应用程序是否按时完成了该帧的工作 

### Jank Type 卡顿类型 

该框架是否出现卡顿现象？如果是，这将显示观察到的卡顿类型。如果不是，类型将为None  

#### **AppJanks**
##### AppDeadlineMissed 

该应用程序的运行时间比预期长，导致卡顿 

##### BufferStuffing 

这更像是一种状态，而不是卡顿。如果应用程序在上一帧呈现之前不断向 SurfaceFlinger 发送新帧，就会发生这种情况。 

内部缓冲区队列填充了尚未呈现的缓冲区，因此得名“缓冲区填充”。队列中的这些额外缓冲区仅一个接一个地呈现，从而导致额外的延迟。 

这还可能导致应用程序没有更多缓冲区可供使用，并且进入出队阻塞等待状态。应用程序执行的实际工作持续时间可能仍在截止日期内，但由于填充的性质，无论应用程序完成工作的速度有多快，所有帧都将至少延迟一个垂直同步。在此状态下，帧仍将平滑，但与较晚的呈现相关的**输入延迟**会增加。

#### **SurfaceFlinger Janks**  

##### SurfaceFlingerCpuDeadlineMissed 

SurfaceFlinger预计将在给定的期限内完成。如果主线程运行的时间超过该时间。 

##### SurfaceFlingerGpuDeadlineMissed 

SurfaceFlinger主线程在CPU上花费的时间+GPU组合时间加在一起比预期的要长。 

##### DisplayHAL 

SurfaceFlinger 完成其工作并将帧按时发送到 HAL，但该帧未在垂直同步上呈现。 

##### PredictionError 

SurfaceFlinger 的调度程序提前计划呈现帧。然而，这种预测有时会偏离实际的硬件垂直同步时间。例如，一帧可能预测当前时间为 20 毫秒。由于估计的偏差，该帧的实际当前时间可能是 23 毫秒。这在 SurfaceFlinger 的调度程序中称为预测错误。调度程序会定期进行自我纠正，因此这种漂移不是永久性的。然而，出于跟踪目的，预测发生漂移的帧仍将被归类为卡顿。


### Prediction type 预测类型 

当 FrameTimeline 收到此帧时，预测是否已过期？如果是，这将显示Expired Prediction 。如果不是，则Valid Prediction。 

### GPU Composition GPU组成 

判断帧是否由 GPU 合成的布尔值。

### Layer Name 图层名称 

呈现框架的层/表面的名称。一些进程将帧更新到多个表面。这里，具有相同标记的多个切片将显示在实际时间轴中。图层名称可以是消除这些切片之间歧义的好方法。

### Is Buffer? 是否是缓冲区 

布尔值，用于判断该帧是否对应于缓冲区或动画。 

## Actual TImeline颜色

| Color 颜色        |                              Image 图像                               | Description 描述                                                                                                                                              |
| :-------------- | :-----------------------------------------------------------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Green 绿色的       |    ![](https://perfetto.dev/docs/images/frametimeline/green.png)    | A good frame. No janks observed  <br>一个好的帧。没有观察到卡顿                                                                                                          |
| Light Green 浅绿色 | ![](https://perfetto.dev/docs/images/frametimeline/light-green.png) | High latency state. The framerate is smooth but frames are presented late, resulting in an increased input latency.  <br>高延迟状态。帧速率平滑，但帧呈现较晚，导致输入延迟增加。       |
| Red 红色的         |     ![](https://perfetto.dev/docs/images/frametimeline/red.png)     | 卡帧，切片所属的进程是卡顿的原因。                                                                                                                                           |
| Yellow 黄色的      |   ![](https://perfetto.dev/docs/images/frametimeline/yellow.png)    | Used only by the apps. The frame is janky but app wasn't the reason, SurfaceFlinger caused the jank.  <br>SurfaceFlinger 导致了卡顿。                             |
| Blue 蓝色的        |    ![](https://perfetto.dev/docs/images/frametimeline/blue.png)     | Dropped frame. Not related to jank. The frame was dropped by SurfaceFlinger, preferring an updated frame over this.  <br>掉帧了。与卡顿无关。该帧被SurfaceFlinger丢弃并跳过了。 |
