不同的人对流畅性(卡顿掉帧)有不同的理解，对卡顿阈值也有不同的感知，所以有必要在开始这个系列文章之前，先把涉及到的内容说清楚，防止出现不同的理解，也方便大家带着问题去看这几篇问题，下面是一些基本的说明

1. 对手机用户来说，卡顿包含了很多场景，比如在 **滑动列表的时候掉帧**、**应用启动白屏过长**、**点击电源键亮屏慢**、**界面操作没有反应然后闪退**、**点击图标没有响应**、**窗口动画不连贯、滑动不跟手、重启手机进入桌面卡顿** 等场景，这些场景跟我们开发人员所理解的卡顿还有点不一样，开发人员会更加细分去分析这些问题，这是开发人员和用户之间的一个认知差异，这一点在处理用户(或者测试人员)的问题反馈的时候尤其需要注意
2. 对开发人员来说，上面的场景包括了 **流畅度**（滑动列表的时候掉帧、窗口动画不连贯、重启手机进入桌面卡顿）、**响应速度**（应用启动白屏过长、点击电源键亮屏慢、滑动不跟手）、**稳定性**（界面操作没有反应然后闪退、点击图标没有响应）这三个大的分类。之所以这么分类，是因为每一种分类都有不太一样的分析方法和步骤，快速分辨问题是属于哪一类很重要
3. 在技术上来说，**流畅度、响应速度、稳定性**（ANR）这三类之所以用户感知都是卡顿，是因为这三类问题产生的原理是一致的，都是由于主线程的 Message 在执行任务的时候超时，根据不同的超时阈值来进行划分而已，所以要理解这些问题，需要对系统的一些基本的运行机制有一定的了解，本文会介绍一些基本的运行机制
4. 流畅性这个系列主要是分析流畅度相关的问题，响应速度和稳定性会有专门的文章介绍，在理解了流畅性相关的内容之后，再去分析响应速度和稳定性问题会事半功倍
5. 流畅性这个系列主要是讲如何使用 Systrace (Perfetto) 工具去分析，之所以 Systrace 为切入点，是因为影响流畅度的因素很多，有 App 自身的原因、也有系统的原因。而 Systrace(Perfetto) 工具可以从一个整机运行的角度来展示问题发生的过程，方便我们去初步定位问题

Systrace (Perfetto) 工具的基本使用如果还不是很熟悉，那么需要优先去补一下上面列出的 [Systrace 基础知识系列](https://www.androidperformance.com/2019/05/26/Android_Systrace_0/)，本文假设你已经熟悉 Systrace(Perfetto)的使用了

# Systrace 的 Frame 颜色是什么意思？

这里的 Frame 标记指的是应用主线程上面那个圈，共有三个颜色，每一帧的耗时不同，则标识的颜色不同

点击这个小圆圈就可以看到这一帧所对应的主线程+渲染线程（会以高亮显示，其他的则变灰显示）

## 绿帧

绿帧是最常见的帧，表示这一帧在一个 Vsync 周期里面完成

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161734002.png)


## 黄帧[](https://www.androidperformance.com/2021/04/24/android-systrace-smooth-in-action-3/#%E9%BB%84%E5%B8%A7)

黄帧表示这一帧耗时超过1个 Vsync 周期，但是小于 2 个 Vsync 周期。黄帧的出现表示这一帧可能存在性能问题，可能会导致卡顿情况出现

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161734786.png)

## 红帧[](https://www.androidperformance.com/2021/04/24/android-systrace-smooth-in-action-3/#%E7%BA%A2%E5%B8%A7)

红帧表示这一帧耗时超过 2 个 Vsync 周期，红帧的出现表示这一帧可能存在性能问题，大概率会导致卡顿情况出现

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161734293.png)


# 没有红帧就没有掉帧？

不一定，判断是否掉帧要看 SurfaceFlinger，而不是看 App ，这部分需要有 [https://www.androidperformance.com/2019/12/15/Android-Systrace-Triple-Buffer/](https://www.androidperformance.com/2019/12/15/Android-Systrace-Triple-Buffer/) 这篇文章的基础

## 出现黄帧但是不掉帧的情况[](https://www.androidperformance.com/2021/04/24/android-systrace-smooth-in-action-3/#%E5%87%BA%E7%8E%B0%E9%BB%84%E5%B8%A7%E4%BD%86%E6%98%AF%E4%B8%8D%E6%8E%89%E5%B8%A7%E7%9A%84%E6%83%85%E5%86%B5)

如上所述，红帧和黄帧都表示这一帧存在性能问题，黄帧表示这一帧耗时超过一个 Vsync 周期，但是由于 Android Triple Buffer（现在的高帧率手机会配置更多的 Buffer）的存在，就算 App 主线程这一帧超过一个 Vsync 周期，也会由于多 Buffer 的缓冲，使得这一帧并不会出现掉帧

## 出现黄帧且掉帧的情况[](https://www.androidperformance.com/2021/04/24/android-systrace-smooth-in-action-3/#%E5%87%BA%E7%8E%B0%E9%BB%84%E5%B8%A7%E4%B8%94%E6%8E%89%E5%B8%A7%E7%9A%84%E6%83%85%E5%86%B5)

这次分析的 Systrace（见附件），就是没有红帧只有黄帧，连续出现两个黄帧，第一个黄帧导致了卡顿，而第二个黄帧则没有

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161735380.png)


# 主线程为何要等待渲染线程？

还是这个 Systrace（附件） 中的情况，第一个疑点处两个黄帧，可以看到第二个黄帧的主线程耗时很久，这时候不能单纯以为是主线程的问题（因为是 Sleep 状态）

如下图所示，是因为前一帧的渲染线程超时，导致这一帧的渲染线程任务在排队等待，如（[https://www.androidperformance.com/2019/11/06/Android-Systrace-MainThread-And-RenderThread/](https://www.androidperformance.com/2019/11/06/Android-Systrace-MainThread-And-RenderThread/)）这篇文章里面的流程，主线程是需要等待渲染线程执行完 syncFrameState 之后 unblockMainThread，然后才能继续。

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161735377.png)


# 为什么一直滑动不松手，就不会卡？

还是这个场景（桌面左右滑动），卡顿是发生在松手之后的，如果一直不松手，那么就不会出现卡顿，这是为什么？

如下图，可以看到，如果不松手，cpu 这里会有一个持续的 Boost，且此时 RenderThread 的任务都跑在 4-6 这三个大核上面，没有跑到小核，自然也不会出现卡顿情况

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161735373.png)


这一段 Boost 的 Timeout 是 120 ms，具体的配置每个机型都不一样，熟悉 PerfLock 的应该知道，这里就不多说了

# 如果不卡，怎么衡量性能好坏？

如果这个场景不卡，那么我们怎么衡量两台不同的机器在这个场景下的性能呢？

可以使用 `adb shell dumpsys gfxinfo` ，使用方法如下

1. 首先确定要测试的包名，到 App 界面准备好操作
2. 执行2-3次 `adb shell dumpsys gfxinfo com.miui.home framestats reset` ，这一步的目的是清除历数据
3. 开始操作（比如使用命令行左右滑动，或者自己用手指滑动）
4. 操作结束后，执行 `adb shell dumpsys gfxinfo com.miui.home framestats` 这时候会有一堆数据输出，我们只需要关注其中的一部分数据即可
5. 重点关注
    1. **Janky frames** ：超过 Vsync 周期的 Frame，不一定出现卡顿
    2. **95th percentile** ：95% 的值
    3. **HISTOGRAM** ： 原始数值
    4. **PROFILEDATA** ：每一帧的详细原始数据

我们拿这个场景，跟 Oppo Reno 5 来做对比，只取我们关注的一部分数据

## 小米 - 90 fps

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161736406.png)


## Oppo - 90 fps

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403161736492.png)

下面是一些对比，可以看到小米在桌面滑动这个场景，性能是要弱于 Oppo 的

1. Janky frames
    1. 小米：27 (35.53%)
    2. Oppo：1 (1.11%)
2. 95th percentile
    1. 小米：18ms
    2. Oppo：5ms

另外 GPU 的数据也比较有趣，小米的高通 865 配的 GPU 要比 Reno 5 Pro 配的 GPU 要强很多，所以 GPU 的数据小米要比 Reno 5 Pro 要好，也可以推断出这个场景的瓶颈在 CPU 而不是在 GPU

# 为什么录屏看不出来卡顿？

可能有下面几种情况

1. 如果使用的手机是大于 60 fps 的，比如小米这个是 90 fps，而录屏的时候选择 60 fps 的录屏，则录屏文件会看不出来卡顿 （使用其他手机录像也会有这个问题）
2. 如果录屏是以高帧率（90fps）录制的，但是播放的时候是使用低帧率（60fps）的设备观看的（小米就是这个情况），也不会看出来卡顿，比如用 90 fps 的规格录制视频，但是在手机上播放的时候，系统会自动切换到 60 fps, 导致看不出来卡顿

# 系列文章

1. [Systrace 流畅性实战 1 ：了解卡顿原理](https://www.androidperformance.com/2021/04/24/android-systrace-smooth-in-action-1/)
2. [Systrace 流畅性实战 2 ：案例分析: MIUI 桌面滑动卡顿分析](https://www.androidperformance.com/2021/04/24/android-systrace-smooth-in-action-2/)
3. [Systrace 流畅性实战 3 ：卡顿分析过程中的一些疑问](https://www.androidperformance.com/2021/04/24/android-systrace-smooth-in-action-3/)

# 附件

附件已经上传到了 Github 上，可以自行下载：[https://github.com/Gracker/SystraceForBlog/tree/master/Android_Systrace_Smooth_In_Action](https://github.com/Gracker/SystraceForBlog/tree/master/Android_Systrace_Smooth_In_Action)

1. xiaomi_launcher.zip : 桌面滑动卡顿的 Systrace 文件，这次案例主要是分析这个 Systrace 文件
2. xiaomi_launcher_scroll_all_the_time.zip : 桌面一直按着滑动的 Systrace 文件
3. oppo_launcher_scroll.zip ：对比文件

# 原文链接

1. [Systrace 流畅性实战 3 ：卡顿分析过程中的一些疑问](https://www.androidperformance.com/2021/04/24/android-systrace-smooth-in-action-3/)
