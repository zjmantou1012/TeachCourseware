---
author: zjmantou
title: （6）- 车载多媒体（一）- 音视频基础知识与MediaPlayer
time: 2023-09-18 周一
tags:
  - Android
  - 车联网
  - MediaPlayer
---
多媒体应用是车载信息娱乐系统的一个重要组成部分，一般包含音视频播放、收音机、相册等。车载应用多媒体系列初步计划分为六篇，这是第一篇。

参考资料  
[视频和视频帧：视频和帧基础知识整理](https://links.jianshu.com/go?to=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F61747783)  
[百度百科 - 声道](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E5%25A3%25B0%25E9%2581%2593%2F2119484) 、 [百度百科 - 量化精度](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E9%2587%258F%25E5%258C%2596%25E7%25B2%25BE%25E5%25BA%25A6) 等  
[管理音频焦点 | Android 开发者 | Android Developers](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Fguide%2Ftopics%2Fmedia-apps%2Faudio-focus%3Fhl%3Dzh_cn)  
[Android音视频开发 - 何俊林](https://links.jianshu.com/go?to=https%3A%2F%2Fbook.douban.com%2Fsubject%2F30364208%2F)  
[MediaPlayer | Android Developers](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Freference%2Fandroid%2Fmedia%2FMediaPlayer%3Fhl%3Dzh_cn)  
[MediaPlayer 概览 | Android 开发者 | Android Developers](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Fguide%2Ftopics%2Fmedia%2Fmediaplayer%3Fhl%3Dzh_cn%23kotlin)

## 1. 音视频基础知识

### 1.1 视频编码

视频编码就是指通过特定的压缩技术，将某个视频格式文件转换成另一种视频格式文件的方式。  
视频编码分为以下两个系列：  
**MPEG系列**：由ISO[国际标准化组织]下属的MPEG[动态影像专家组]开发。视频编码方面主要是MPEG1（VCD使用）、MPEG2（DVD使用）、MPEG4（DVD RIP使用的都是它的变种，如DivX、XviD等）、MPEG4 AVC（目前最常用）。其还有音频编码方面，主要有MPEG Audio Layer1/2、MPEG Audio Layer 3（MP3）、MPEG-2 AAC、MPEG4-AAC等。DVD音频没有采用MPEG。  
**H.26X系列**：由ITU[国际电传视讯联盟]主导，侧重网络传输，但只有视频编码。包括H.261、H.262、H.263、H.263+、H.263++、H.264（与MPEG4 AVC合作的产物）。

### 1.2 音频编码

常见的音频编码格式有AAC、MP3、AC3。  
**AAC**：AAC，全称Advanced Audio Coding，是一种专为声音数据设计的文件压缩格式。与MP3不同，它采用了全新的算法进行编码，更加高效，具有更高的“性价比”。利用AAC格式，可使人感觉声音质量没有明显降低的前提下，更加小巧。苹果ipod、诺基亚手机支持AAC格式的音频文件。**优点：**相较于mp3，AAC格式的音质更佳，文件更小。**不足：**AAC属于有损压缩的格式，与时下流行的APE、FLAC等无损格式相比音质存在“本质上”的差距。加之，传输速度更快的USB3.0和16G以上大容量MP3正在加速普及，也使得AAC头上“小巧”的光环不复存在。  
**MP3**：MP3是一种音频压缩技术，其全称是动态影像专家压缩标准音频层面3（Moving Picture Experts Group Audio Layer III），简称为MP3。它被设计用来大幅度地降低音频数据量。利用 MPEG Audio Layer 3 的技术，将音乐以1:10 甚至 1:12 的压缩率，压缩成容量较小的文件，而对于大多数用户来说重放的音质与最初的不压缩音频相比没有明显的下降。  
MP3是利用人耳对高频声音信号不敏感的特性，将时域波形信号转换成频域信号，并划分成多个频段，对不同的频段使用不同的压缩率，对高频加大压缩比（甚至忽略信号）对低频信号使用小压缩比，保证信号不失真。这样一来就相当于抛弃人耳基本听不到的高频声音， [1] 只保留能听到的低频部分，从而将声音用1∶10甚至1∶12的压缩率压缩。由于这种压缩方式的全称叫MPEG Audio Player3，所以人们把它简称为MP3。  
根据MPEG规范的说法，MPEG-4中的AAC（Advanced audio coding）将是MP3格式的下一代。  
最高参数的MP3(320Kbps)的音质较之CD的,FLAC和APE无损压缩格式的差别不多，其优点是压缩后占用空间小，适用于移动设备的存储和使用。  
**AC3**：全称Audio Coding3（音频编码3）是杜比数码（Dolby Digital）的同义词，杜比数码是一种高级音频压缩技术，它最多可以对6个比特率最高为448kbps的单独声道进行编码。杜比AC-3提供的环绕声系统由5个全频域声道和1个超低音声道组成，被称为5.1声道。5个声道包括左前、中央、右前、左后、右后。低音声道主要提供一些额外的低音信息，使一些场景，如爆炸、撞击等声音效果更好。

### 1.3 多媒体播放组件

在Android系统中多媒体播放组件包含MediaPlayer、MediaCodec、OMX、StageFright、AudioTrack 。

- **MediaPlayer**：将系统提供的解码器封装后提供给应用开发者使用音视频播放组件，一般支持多种多媒体格式。
- **MediaCodec**：音视频编解码 。
- **OMX**：多媒体部分采用的编解码标准。
- **Stagefright**：Stagefright是位于原生层的媒体播放引擎，内置了基于软件的编解码器，可用于处理热门媒体格式。它是用来替代之前OpenCore框架，主要做了一层OMX层，仅仅对OpenCore的omx-component部分做了引用。Stagefright是在MediaPlayerService这一层加入的，和OpenCore是并列的。StageFright在Android中以共享库的形式存在（libstagefright.so），其中的Module - NuPlayer/AwesomePlayer可用来播放音视频。NuPlayer/AwesomePlayer提供了许多的API，可以让上层的应用程序（Java/JNI）调用。
- **AudioTrack**：音频播放组件，与MediaPlayer不同的是，AudioTrack仅支持非压缩编码（PCM）的音频。

### 1.4 音视频中的专业术语

### 1.4.1 帧率

**帧率**（frame rate）是用于测量显示帧数的度量。测量单位为“每秒显示帧数”（**frame per second**，**FPS**）或“赫兹”，一般来说FPS用于描述视频、电子绘图或游戏每秒播放多少帧。  
每秒的帧数(fps)或者说帧率表示图形处理器处理场时每秒钟能够更新的次数。高的帧率可以得到更流畅、更逼真的动画。一般来说30fps就是可以接受的，但是将性能提升至60fps则可以明显提升交互感和逼真感，但是一般来说超过75fps一般就不容易察觉到有明显的流畅度提升了。如果帧率超过屏幕刷新率只会浪费图形处理的能力，因为监视器不能以这么快的速度更新，这样超过刷新率的帧率就浪费掉了。

### 1.4.2 分辨率

视频分辨率，是用于度量图像内数据量多少的一个参数，通常表示成ppi。  
视频的320X180是指它在横向和纵向上的有效像素，窗口小时ppi值较高，看起来清晰；窗口放大时，由于没有那么多有效像素填充窗口，有效像素ppi值下降，就模糊了。

### 1.4.3 刷新率

刷新率是指电子束对屏幕上的图像重复扫描的次数。刷新率越高，所显示的图像（画面）稳定性就越好。  
刷新率分为垂直刷新率和水平刷新率，一般提到的刷新率通常指垂直刷新率。垂直刷新率表示屏幕的图像每秒钟重绘多少次，也就是每秒钟屏幕刷新的次数，以Hz（赫兹）为单位。刷新率越高越好，图像就越稳定，图像显示就越自然清晰，对眼睛的影响也越小。刷新频率越低，图像闪烁和抖动的就越厉害，眼睛疲劳得就越快。一般来说，如能达到80Hz以上的刷新频率就可完全消除图像闪烁和抖动感，眼睛也不会太容易疲劳。

### 1.4.4 编码格式

编码的目的就是指通过特定的压缩技术，将某个视频格式的文件转换成另一种视频格式文件的方式，主要目标是压缩数据量。常用的编码格式有MPEG和H.26X两种。

### 1.4.5 封装格式

封装格式（也叫容器），就是将已经编码压缩好的视频轨和音频轨按照一定的格式放到一个文件中，也就是说仅仅是一个外壳。常见的封装格式有：  
**AVI**：微软在90年代初创立的封装标准，是当时为对抗quicktime格式（mov）而推出的，只能支持固定CBR恒定比特率编码的声音文件。  
**FLV**：针对于h.263家族的格式。  
**MKV**：万能封装器，有良好的兼容和跨平台性、纠错性，可带 外挂字幕。  
**MOV**：MOV是Quicktime封装。  
**MP4**：主要应用于mpeg4的封装 。  
**RM/RMVB**：Real Video，由RealNetworks开发的应用于rmvb和rm 。  
**TS/PS**：PS封装只能在HDDVD原版。  
**WMV**：微软推出的，作为市场竞争。

### 1.4.6 码率

比特率又称“二进制位速率”，俗称“码率”。表示单位时间内传送比特的数目。用于衡量数字信息的传送速度，常写作bit/sec。根据每帧图像存储时所占的比特数和传输比特率，可以计算数字图像信息传输的速度。码率越高，消耗的带宽越多。  
文件大小(b) = 码率(b/s) X 时长(s)

### 1.4.7 画质&码率

通常我们有一个**错误的认识**，码率越大画质越好！实际上视频质量和码率、编码算法都有关系。

### 1.4.8 DTS&PTS

**DTS**：解码时间标签（Decoding Time Stamp）。主要用于标识读入内存中的比特流在什么时候开始送入解码器中进行解码。  
**PTS**：演示时间标签（Presentation Time Stamp）。主要用于度量解码后的视频帧什么时候被显示出来。

### 1.4.9 YUV&RGB

**YUV**：是一种颜色编码方法。YUV是编译true-color颜色空间（color space）的种类，Y'UV, YUV, YCbCr，YPbPr等专有名词都可以称为YUV，彼此有重叠。“Y”表示明亮度（Luminance或Luma），也就是灰阶值，“U”和“V”表示的则是色度（Chrominance或Chroma），作用是描述影像色彩及饱和度，用于指定像素的颜色。  
**RGB**：RGB色彩模式是工业界的一种颜色标准，是通过对红(R)、绿(G)、蓝(B)三个颜色通道的变化以及它们相互之间的叠加来得到各式各样的颜色的，RGB即是代表红、绿、蓝三个通道的颜色，这个标准几乎包括了人类视力所能感知的所有颜色，是运用最广的颜色系统之一。

### 1.4.10 视频帧&音频帧

常见的视频帧有I、P、B帧等。

- **I帧**，**Intra Picture**，又称**帧内编码帧**，俗称**关键帧**。一般来说**I帧**不需要依赖前后帧信息，可独立进行解码。
- **P帧**，**predictive-frame**，又称**前向预测编码帧**，也有**帧间预测编码帧**。顾名思义，**P帧**需要依赖前面的**I帧**或者**P帧**才能进行编解码，因为一般来说，**P帧**存储的是当前帧画面与前一帧（前一帧可能是**I帧**也可能是**P帧**）的差别，较专业的说法是**压缩了时间冗余信息，或者说提取了运动特性**。**P帧**的压缩率约在20左右，几乎所有的H264编码流都带有大量的**P帧**。
- **B帧**，**bi-directional interpolatedprediction frame**，又称**双向预测内插编码帧**，简称**双向预测编码帧**。**B帧**非常特殊，它存储的是本帧与前后帧的差别，因此带有B帧的视频在解码时的逻辑会更复杂些，CPU开销会更大。因此，不是所有的视频都带有**B帧**，不过，**B帧**的压缩率能够达到50甚至更高，在压缩率指标上还是很可观的。

音频帧的概念没有视频帧那么清晰，音频帧与编码格式有关：

- 对于PCM（脉冲编码调制，非压缩编码）来说，它不需要帧的概念，根据采样率和采样精度就可以播放。
- AMR帧比较简单，它规定每20ms的音频就是1帧，每1帧的音频都是独立的，有可能采用不同的编码算法以及不同的编码参数。
- MP3帧较为复杂一些，包含了更多信息，比如采样率、比特率等各种参数。具体如下：音频数据帧个数由文件大小和帧长决定，每一帧的长度可能不固定，也可能固定，由比特率决定，每一帧又分为帧头和数据实体两部分，帧头记录了MP3的比特率、采样率、版本等信息，每一帧之间相互独立。

### 1.4.11 量化精度

量化精度，是指可以将模拟信号分成多少个等级，量化精度越高，越接近原波形。

### 1.4.12 采样率

采样频率，也称为采样速度或者采样率，定义了单位时间内从连续信号中提取并组成离散信号的采样个数，它用赫兹（Hz）来表示。采样频率的倒数是采样周期或者叫做采样时间，它是采样之间的时间间隔。通俗的讲采样频率是指计算机单位时间内能够采集多少个信号样本。

### 1.4.13 声道

声道(Sound Channel) 是指声音在录制或播放时在不同空间位置采集或回放的相互独立的音频信号，所以声道数也就是声音录制时的音源数量或回放时相应的扬声器数量。  
**立体声道**：单声道缺乏对声音的位置定位，而立体声技术则彻底改变了这一状况。声音在录制过程中被分配到两个独立的声道，从而达到了很好的声音定位效果。这种技术在音乐欣赏中显得尤为有用，听众可以分辨出各种乐器来自的方向，从而使音乐更富想象力，更加接近于临场感受。立体声技术广泛运用于自Sound Blaster Pro以后的大量声卡，成为了影响深远的一个音频标准。时至今日，立体声依然是许多产品遵循的技术标准。  
**4声道**：四声道环绕规定了4个发音点：前左、前右，后左、后右，听众则被包围在这中间。同时还建议增加一个低音音箱，以加强对低频信号的回放处理(这也就是如今4.1声道音箱系统广泛流行的原因)。就整体效果而言，四声道系统可以为听众带来来自多个不同方向的声音环绕，可以获得身临各种不同环境的听觉感受，给用户以全新的体验。如今四声道技术已经广泛融入于各类中高档声卡的设计中，成为未来发展的主流趋势。  
**5.1声道**：5.1声道已广泛运用于各类传统影院和家庭影院中，一些比较知名的声音录制压缩格式，譬如杜比AC-3（Dolby Digital）、DTS等都是以5.1声音系统为技术蓝本的，其中“.1”声道，则是一个专门设计的超低音声道，这一声道可以产生频响范围20～120Hz的超低音。其实5.1声音系统来源于4.1环绕，不同之处在于它增加了一个中置单元。这个中置单元负责传送低于80Hz的声音信号，在欣赏影片时有利于加强人声，把对话集中在整个声场的中部，以增加整体效果。相信每一个真正体验过Dolby AC-3音效的朋友都会为5.1声道所折服。

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670159878400-164bd023-70a8-4d5c-a172-ae0ff5cf4121.webp)

**7.1声道**：7.1声道系统的作用简单来说就是在听者的周围建立起一套前后声场相对平衡的声场,不同于5.1声道声场的是,它在原有的基础上增加了后中声场声道,同时它也不同于普通6.1声道声场,因为 [2] 7.1声道有双路后中置,而这双路后中置的最大作用就是为了防止听者因为没有坐在皇帝位而在听觉上产生声场的偏差。

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670159889693-5cf64545-6b4e-4531-af85-258da6de41a1.webp)

## 2.系统播放器 - MediaPlayer

`MediaPlayer`是Android系统中的一个多媒体播放类，通过它能控制音视频流或本地音视频资源的播放过程。在车载音视频应用开发中，许多时候我们会直接采用MediaPlayer，当然可以使用ExoPlayer，这个挖个坑以后再讲。

### 2.1 MediaPlayer的状态与生命周期

音频/视频文件和流的播放控制作为状态机进行管理。下图显示了由支持的播放控制操作驱动的 MediaPlayer 对象的生命周期和状态（**这张图很重要，使用MediaPlayer开发音视频应用时，务必参考**）。  
椭圆形表示 MediaPlayer 对象可能驻留的状态。弧表示驱动对象状态过渡的播放控制操作。有两种类型的弧。具有单个箭头的弧表示同步方法调用，而具有双箭头的弧表示异步方法调用。

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670159942243-5d9e31f0-0adc-4a11-ba02-f4dd954dc237.webp)

从此状态图中，可以看到 MediaPlayer 对象具有以下状态：

- **Idel 和 End 状态**  
    当创建`MediaPlayer`实例或调用`reset()`后，它处于**Idle（空闲/就绪）**状态；调用`release()`之后，它处于**End（结束）**状态。**在Idle和End之间的状态就是**`**MediaPlayer**`**对象的生命周期**。
- **Error 状态**  
    在创建一个新的`MediaPlayer`的实例或调用`reset()`后，此时`MediaPlayer`尚处于**Idle**状态，如果此时调用`getCurrentPosition()`、`getDuration()`、`getVideoHeight()`、`getVideoWidth()`、`setAudioAttributes(android.media.AudioAttributes)`、`setLooping(boolean)`、`setVolume(float, float)`、`pause()`、`start()`、`stop()`、`seekTo(long, int)`、`prepare()`、`prepareAsync()`程序就会出错，并触发`setOnErrorListener`设定的`OnErrorListener.onError`，然后`MediaPlayer`的生命周期就会进入**Error**状态。  
    通常，某些播放控制操作可能会由于各种原因而失败，例如音频/视频格式不受支持、音频/视频交错不良、分辨率过高、流超时等。因此，在这些情况下，错误报告和恢复是一个重要的问题。有时，由于编程错误，还可能发生在无效状态下调用播放控制操作的情况。在所有这些错误条件下，都会触发`setOnErrorListener`设定的`OnErrorListener.onError`，`MediaPlayer`的生命周期同样会进入**Error**状态。  
    为了重新使用`MediaPlayer`，调用`reset()`可以将`MediaPlayer`恢复到**Idle状态**。
- **End 状态**  
    `MediaPlayer`会占用宝贵的系统资源。因此，应该始终采取额外的预防措施，确保 `MediaPlayer`对象保留的时间不会过长。调用 `release()`可以确保分配给`MediaPlayer`的系统资源得到释放。此时`MediaPlayer`的生命周期则会进入**End（结束）**状态。  
    **当**`MediaPlayer`**处于End状态时，它就不能再使用了，也无法再进入其它的生命状态**。
- **Initialized 状态**  
    处于**Idle**状态时，调用`setDataSource(java.io.FileDescriptor)`、`setDataSource(java.lang.String)`、`setDataSource(android.content.Context, android.net.Uri)`、`setDataSource(java.io.FileDescriptor, long, long)`、`setDataSource(android.media.MediaDataSource)`中的任意方法，`MediaPlayer`会进入**Initialized（已初始化）**状态。  
    如果在非**Idle**状态调用`setDataSource()`，会引发**IllegalStateException。**  
    重载`setDataSource()`，需要抛出**IllegalArgumentException**、**IOException**。
- **Prepared 状态**  
    `MediaPlayer`对象必须先进入**Prepared（已准备)**状态，然后才能开始播放。  
    有两种途径到达**Prepared状态**。一种是调用prepare()，这是一种同步方法，由于**prepare**本身是耗时操作，虽然有时候它执行的很快，但也不要在主线程执行它，否则可能导致ANR。另一种方式是调用`prepareAsync()`，它是一种异步方法，可以在主线程中执行。调用`prepareAsync()`并不会立即进入**Prepared状态**，而是先进入**Preparing状态，**最后到达**Prepared状态**。需要注意**Preparing**是一种瞬间状态，存在时间很短**。**通过注册`setOnPreparedListener(android.media.MediaPlayer.OnPreparedListener)`可以监听`MediaPlayer`的**Prepare**状态。  
    在**Prepared状态**下，`Volume`、`screenOnWhilePlaying` 、`Looping`等属性已经可以通过调用相应的set 方法来调整了。  
    如果在非**Initialized**状态调用`prepare()` / `prepareAsync()`，会引发**IllegalStateException。**
- **Started状态**  
    在播放之前必须调用`start()`并成功返回，此时`MediaPlayer`状态进入Started（已启动）状态，通过注册`setOnBufferingUpdateListener(android.media.MediaPlayer.OnBufferingUpdateListener)`可以保持跟踪音视频流的buffering（缓冲）status。  
    当`MediaPlayer`处于**Started状态**时，可以通过调用`seekTo(long, int)`来调整播放位置。尽管异步调用会立即返回，但实际的寻道操作可能需要一段时间才能完成，特别是对于正在流式传输的音频/视频。当实际的寻道操作完成时，如果事先注册了`setOnSeekCompleteListener(android.media.MediaPlayer.OnSeekCompleteListener)`，则内部播放器引擎会回调OnSeekComplete.onSeekComplete()。  
    此外，可以通过调用`getCurrentPosition()`来检索实际的当前播放位置，这对于需要跟踪播放进度的应用程序（如 Music 播放器）非常有用。
- **Paused状态**  
    在音视频播放过程中调用`pause()`方法，`MediaPlayer`的状态从**Started状态**进入**Paused（已暂停）状态**，这个过程是瞬间的且在播放器内部是异步的。反之，从**Paused**通过调用`start()`返回**Started**也是同样的。在状态更新并调用`isPlaying()`方法前，将有一些耗时，对流数据可能需要耗费数秒。  
    当`MediaPlayer`处于**Paused**，此时调用`seekTo(long, int)`来调整播放位置时，如果数据流具有视频并且请求的位置有效，则将显示一个视频帧。
- **Stopped状态**  
    当调用`stop()`方法时，`MediaPlayer`无论是处于**Started**、**Paused**、**Prepared**还是**PlaybackCompleted**中哪种状态，都将进入**Stoped（已停止）状态**。一旦进入**Stoped状态**，playback将不能开始，直到调用`prepare()` / `prepareAsync()`，且处于**Prepared状态**才可以开始。
- **PlaybackCompleted状态**  
    当MediaPlayer播放到数据流末尾时，一次播放完成。如果事先`setLooping(true)`，`MediaPlayer`依然处于**Started状态**，并重新开始播放。如果实现`setLooping(false)`，如果事先注册了`setOnCompletionListener(android.media.MediaPlayer.OnCompletionListener)`，就会回调 OnCompletion.onCompletion()，表示`MediaPlayer`进入**PlaybackCompleted状态**。处于**PlaybackCompleted**时调用`start()`方法会重启播放器从头开始播放数据，此时状态进入**Started。**  
    当`MediaPlayer`处于**PlaybackCompleted**，此时调用`seekTo(long, int)`来调整播放位置时，如果数据流具有视频并且请求的位置有效，则将显示一个视频帧。

### 2.2 MediaPlayer 简略使用

#### 2.2.1 权限声明

在开始使用`MediaPlayer`开发应用之前，还需要在manifest中添加适当的声明，这样才能使用相关功能。

- **互联网权限** - 如果需要使用MediaPlayer播放基于网络的内容，则必须申请网络访问权限。

```
<uses-permission android:name="android.permission.INTERNET" /> 
```

- **唤醒锁定权限** - 如果播放器应用需要防止屏幕变暗或处理器进入休眠状态，或者要使用MediaPlayer.setScreenOnWhilePlaying() 或 MediaPlayer.setWakeMode() 方法，则必须申请此权限。

```
<uses-permission android:name="android.permission.WAKE_LOCK" /> 
```

#### 2.2.2 播放资源

`MediaPlayer`支持多种不同的媒体源，例如：

- 本地资源（打包在应用中的资源）
- 内部 URI，例如，从ContentProvider那获取的 URI
- 外部网址（流式传输）  
    以下示例展示了如何播放作为本地原始资源（保存在应用的 res/raw/ 目录中）提供的音频：

```kotlin
var mediaPlayer: MediaPlayer? = MediaPlayer.create(context, R.raw.sound_file_1) 
mediaPlayer?.start() // 不需要调用prepare，因为create中已经替我们做好了 
```

在本例中，“原始”资源是指系统不会尝试以任何特定方式解析的文件。不过，该资源的内容不应为原始音频。它应该是采用某种支持的格式且经过适当编码和格式化的媒体文件。  
播放系统中本地可用的 URI（例如，可以通过ContentProvider获取）中的内容方法如下：

```kotlin
val resUri: Uri = .... // 本地uri
val mediaPlayer: MediaPlayer? = MediaPlayer().apply {
    setAudioStreamType(AudioManager.STREAM_MUSIC)
    setDataSource(applicationContext, resUri)
    prepare()
    start()
}
```

通过 HTTP 流式传输并播放远程网址上的内容如下所示：

```kotlin
val url = "http://........" // 网络url
val mediaPlayer: MediaPlayer? = MediaPlayer().apply {
    setAudioStreamType(AudioManager.STREAM_MUSIC)
    setDataSource(url)
    prepare() // 耗时操作，可能导致ANR
    start()
}
```

注意 ：  
如果是播放在线媒体文件，则该文件必须能够进行渐进式下载。  
使用 setDataSource() 时，必须捕获或传递 IllegalArgumentException 和 IOException，因为引用的文件可能并不存在。

### 2.3 后台播放

如果希望即使当应用未在屏幕上显示时，应用仍会在后台播放媒体内容，则必须启动一个Service并由此控制 `MediaPlayer` 实例。

### 2.4 唤醒锁定

当设计在后台播放媒体内容的应用时，**手机等移动设备可能会在Service运行时进入休眠状态，车载设备则无法确定，目前尚无统一的规范，每个主机厂的策略可能都不相同。**  
在原生系统中通过调用`setWakeMode()`可以保持唤醒锁定，完成该操作后，`MediaPlayer` 会在播放时保持指定的锁定状态，并在暂停或停止播放时释放锁定。

```kotlin
mediaPlayer = MediaPlayer().apply { 
    // 省略其它代码 
    setWakeMode(applicationContext, PowerManager.PARTIAL_WAKE_LOCK) 
} 
```

不过，播放流媒体时，还需要保持wifi的锁定状态，否则可能出现wifi中断的现象。

```kotlin
val wifiManager = getSystemService(Context.WIFI_SERVICE) as WifiManager 
val wifiLock: WifiManager.WifiLock = wifiManager.createWifiLock(WifiManager.WIFI_MODE_FULL, "mylock") 
wifiLock.acquire() 
```

当不再需要网络时，通过如下方式释放该锁定。

```kotlin
wifiLock.release() 
```

### 2.5 数字版权管理 (DRM)

从 Android 8.0（API 26）开始，MediaPlayer 包含支持播放受 DRM 保护的资料的 API。这些 API 与由 MediaDrm 提供的低级别 API 类似，但前者是在较高级别运行，并且不会公开底层提取器、DRM 和加密对象。车载多媒体中出现DRM的情况可能不多（我还没遇到过），如有需要可以参考Android官方的文档：[MediaPlayer 概览 | Android 开发者 | Android Developers](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Fguide%2Ftopics%2Fmedia%2Fmediaplayer%3Fhl%3Dzh_cn%23kotlin)

## 3. 管理音频焦点

以下内容选自[管理音频焦点 | Android 开发者 | Android Developers](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Fguide%2Ftopics%2Fmedia-apps%2Faudio-focus%3Fhl%3Dzh_cn) 有增删。

两个或两个以上的 Android 应用可同时向同一输出流播放音频。系统会将所有音频流混合在一起。虽然这是一项出色的技术，但却会给用户带来很大的困扰。为了避免所有音乐应用同时播放，Android 引入了“音频焦点”的概念。 一次只能有一个应用获得音频焦点。  
所以当多媒体应用需要输出音频时，它需要请求音频焦点，顺利获取后，才可以播放声音。当其它应用请求音频焦点时，Android系统会根据内部仲裁表中定义的优先级决定，该应用能否获取音频焦点。例如：在用户使用蓝牙电话应用时，该应用会获取音频焦点，收音机等音频优先级较低的应用就会失去音频焦点。

音频焦点的管理在Android系统中不是强制的，即使应用失去音频焦点，也是可以输出音频的。但是在车载多媒体应用开发时**务必不能这么做**，因为多数驾驶员会同时使用收音机和地图导航，如果导航提示音被收音机的音频压制，就极有可能造成驾驶偏航甚至是交通事故！！

### 3.1 音频焦点管理准则

多媒体应用一般会遵守以下的准则来管理音频焦点：

- 在即将开始播放之前调用 `requestAudioFocus()`，并验证调用是否返回 `AUDIOFOCUS_REQUEST_GRANTED`。
- 在其他应用获得音频焦点时，停止或暂停播放，或降低音量。
- 播放停止后，放弃音频焦点。

### 3.2 Android 8.0 及更高版本中的音频焦点

因为现有车载IVI系统绝大多数已经升级到Android 9.0甚至是更高的版本，所以Android 8.0之前的音频焦点获取方式，本文不再介绍。

从 Android 8.0（API 26）开始，调用`requestAudioFocus()`时，必须提供`AudioFocusRequest`参数。要释放音频焦点，请调用 `abandonAudioFocusRequest()` 方法，该方法也接受 `AudioFocusRequest` 作为参数。在**请求和放弃焦点**时，应使用相同的 `AudioFocusRequest` 实例。要创建 `AudioFocusRequest`，请使用 `AudioFocusRequest.Builder`。由于焦点请求始终必须指定请求的类型，因此此类型会包含在构建器的构造函数中。使用构建器的方法来设置请求的其他字段。  
`FocusGain` 字段为必需字段；所有其他字段均为可选字段。

|   |   |
|---|---|
|方法|备注|
|setFocusGain()|每个请求中都必须包含此字段。此字段的值与 Android 8.0 之前的 requestAudioFocus() 调用中所使用的 durationHint 值相同：AUDIOFOCUS_GAIN、AUDIOFOCUS_GAIN_TRANSIENT、AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK 或 AUDIOFOCUS_GAIN_TRANSIENT_EXCLUSIVE。|
|setAudioAttributes()|AudioAttributes 描述了应用的用例。系统会在应用获得和失去音频焦点时查看这些属性。这些属性取代了音频流类型的概念。在 Android 8.0（API 26）及更高版本中，弃用了除音量控制以外的所有操作的音频流类型。在焦点请求中使用与音频播放器中相同的属性（如此表下面的示例中所示）。首先使用 AudioAttributes.Builder 指定属性，然后使用此方法将属性分配给请求。如果未指定，则 AudioAttributes 默认为 AudioAttributes.USAGE_MEDIA。|
|setWillPauseWhenDucked()|当其他应用使用 AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK 请求焦点时，持有焦点的应用通常不会收到 onAudioFocusChange() 回调，因为系统可以自行降低音量。如果需要暂停播放而不是降低音量，请调用 setWillPauseWhenDucked(true)，然后创建并设置 OnAudioFocusChangeListener，具体如自动降低音量中所述。|
|setAcceptsDelayedFocusGain()|当焦点被其他应用锁定时，对音频焦点的请求可能会失败。此方法可实现延迟获取焦点，即在焦点可用时异步获取焦点。请注意，要使“延迟获取焦点”起作用，必须在音频请求中指定 AudioManager.OnAudioFocusChangeListener，因为应用必须收到回调才能知道自己获取了焦点。|
|setOnAudioFocusChangeListener()|只有在请求中还指定了 willPauseWhenDucked(true) 或 setAcceptsDelayedFocusGain(true) 时，才需要 OnAudioFocusChangeListener。有两个方法可以设置监听器：一个带处理程序参数，一个不带。处理程序是运行监听器的线程。如果未指定处理程序，则会使用与主 Looper 关联的处理程序。|

以下示例展示了如何使用 `AudioFocusRequest.Builder` 构建 `AudioFocusRequest` 来请求和放弃音频焦点

```kotlin
audioManager = getSystemService(Context.AUDIO_SERVICE) as AudioManager
focusRequest = AudioFocusRequest.Builder(AudioManager.AUDIOFOCUS_GAIN).run {
    setAudioAttributes(AudioAttributes.Builder().run {
        setUsage(AudioAttributes.USAGE_GAME)
        setContentType(AudioAttributes.CONTENT_TYPE_MUSIC)
        build()
    })
    setAcceptsDelayedFocusGain(true)
    setOnAudioFocusChangeListener(afChangeListener, handler)
    build()
}
mediaPlayer = MediaPlayer()
val focusLock = Any()

var playbackDelayed = false
var playbackNowAuthorized = false

// ...
val res = audioManager.requestAudioFocus(focusRequest)
synchronized(focusLock) {
    playbackNowAuthorized = when (res) {
        AudioManager.AUDIOFOCUS_REQUEST_FAILED -> false
        AudioManager.AUDIOFOCUS_REQUEST_GRANTED -> {
            playbackNow()
            true
        }
        AudioManager.AUDIOFOCUS_REQUEST_DELAYED -> {
            playbackDelayed = true
            false
        }
        else -> false
    }
}

// ...
override fun onAudioFocusChange(focusChange: Int) {
    when (focusChange) {
        AudioManager.AUDIOFOCUS_GAIN ->
            if (playbackDelayed || resumeOnFocusGain) {
                synchronized(focusLock) {
                    playbackDelayed = false
                    resumeOnFocusGain = false
                }
                playbackNow()
            }
        AudioManager.AUDIOFOCUS_LOSS -> {
            synchronized(focusLock) {
                resumeOnFocusGain = false
                playbackDelayed = false
            }
            pausePlayback()
        }
        AudioManager.AUDIOFOCUS_LOSS_TRANSIENT -> {
            synchronized(focusLock) {
                resumeOnFocusGain = true
                playbackDelayed = false
            }
            pausePlayback()
        }
        AudioManager.AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK -> {
            // ... 暂停或回避取决于你的应用程序
        }
    }
}
```

### 3.3 自动降低音量

在 Android 8.0（API 26）中，当其他应用使用 `AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK` 请求焦点时，系统可以在不调用应用的 `onAudioFocusChange`() 回调的情况下降低和恢复音量。  
虽然自动降低音量的行为对于音乐和视频播放应用来说是可接受的，但在播放语音内容时（例如在听书应用）就没什么用处了。在这种情况下，应用应该暂停播放。  
如果希望应用在被要求降低音量时暂停播放，应该创建包含 `onAudioFocusChange`() 回调方法的 `OnAudioFocusChangeListener`，该回调方法可以实现所需的暂停/恢复行为。 调用 `setOnAudioFocusChangeListener`() 来注册监听器，然后调用 `setWillPauseWhenDucked(true)` 告诉系统使用的回调，而不是执行自动降低音量。

### 3.4 延迟获取焦点

在有些情况下，系统不能批准对音频焦点的请求，因为焦点被其他应用“锁定”了，例如在通话过程中。在这种情况下，`requestAudioFocus`() 会返回 `AUDIOFOCUS_REQUEST_FAILED`。在这种情况下，应用将不会播放音频，因为它未获得焦点。  
方法`setAcceptsDelayedFocusGain(true)`可让应用异步处理焦点请求。设置此标记后，在焦点锁定时发出的请求会返回`AUDIOFOCUS_REQUEST_DELAYED`。当锁定音频焦点的情况不再存在时（例如当通话结束时），系统会批准待处理的焦点请求，并调用`onAudioFocusChange`()来通知应用。  
为了处理“延迟获取焦点”，必须创建包含`onAudioFocusChange`()回调方法的`OnAudioFocusChangeListener`，该回调方法会通过调用 `setOnAudioFocusChangeListener`()来实现所需行为并注册监听器。

### 3.5 响应音频焦点更改

当应用获得音频焦点后，它必须能够在其他应用为自己请求音频焦点时释放该焦点。出现这种情况时，应用会收到对`AudioFocusChangeListener`中的`onAudioFocusChange`()方法的调用，该方法是应用调用`requestAudioFocus`()时指定的。传递给`onAudioFocusChange`()的`focusChange`参数表示所发生的更改类型。它对应于获取焦点的应用所使用的持续时间提示。应用应该做出适当的响应。

#### **3.5.1 暂时性失去焦点**

如果焦点更改是暂时性的`AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK`或`AUDIOFOCUS_LOSS_TRANSIENT`，应用应该降低音量（如果不依赖于自动降低音量）或暂停播放，否则保持相同的状态。在暂时性失去音频焦点时，应该继续监控音频焦点的变化，并准备好在重新获得焦点后恢复正常播放。当抢占焦点的应用放弃焦点时，收到一个回调`AUDIOFOCUS_GAIN`。此时，可以将音量恢复到正常水平或重新开始播放。

#### **3.5.2 永久性失去焦点**

如果是永久性失去音频焦点 (`AUDIOFOCUS_LOSS`)，则其他应用会播放音频。您的应用应立即暂停播放，因为它不会收到`AUDIOFOCUS_GAIN`回调。要重新开始播放，用户必须执行明确的操作，例如在通知或应用界面中按播放传输控件。  
以下代码段展示了如何实现`OnAudioFocusChangeListener`及其`onAudioFocusChange`()回调。请注意这里使用`Handler`延迟了对永久性失去音频焦点的停止回调。

```kotlin
private val handler = Handler()
private val afChangeListener = AudioManager.OnAudioFocusChangeListener { focusChange ->
    when (focusChange) {
          AudioManager.AUDIOFOCUS_LOSS -> {
                // 永久性失去音频焦点，立即暂停播放
                mediaController.transportControls.pause()
                // 等待30秒后停止播放
                handler.postDelayed(delayedStopRunnable, TimeUnit.SECONDS.toMillis(30))
          }
          AudioManager.AUDIOFOCUS_LOSS_TRANSIENT -> {
                // 暂停播放
          }
          AudioManager.AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK -> {
                // 降低音量，继续播放
          }
          AudioManager.AUDIOFOCUS_GAIN -> {
                // 再次被授予音频焦点，将音量调至正常，必要时重新开始播放
          }
   }
}

private var delayedStopRunnable = Runnable {
        mediaController.transportControls.stop()
}
```

为了确保在用户重新开始播放时不会触发延迟停止，请调用 `mHandler.removeCallbacks(mDelayedStopRunnable)`来响应任何状态变化。例如，在回调的`onPlay()`、`onSkipToNext(`) 等中调用 `removeCallbacks`()。此外，在清理服务使用的资源时，也应该在服务的 `onDestroy`() 回调中调用此方法。

### 3.6 AudioFocus 常用常量

|   |   |
|---|---|
|常量|备注|
|**AUDIOFOCUS_NONE**|没有获得、丢失音频焦点，也没有请求音频焦点。|
|**AUDIOFOCUS_GAIN**|用于请求持续时间未知的音频焦点。此参数会触发其他监听器的AudioManager.AUDIOFOCUS_LOSS。|
|**AUDIOFOCUS_GAIN_TRANSIEN**|用于请求短暂性音频焦点。此参数会触发其他监听器的AudioManager.AUDIOFOCUS_LOSS_TRANSIENT。|
|**AUDIOFOCUS_GAIN_TRANSIENT_EXCLUSIVE**|用于申请短暂性音频焦点，一般是录音或者语音识别，此参数会触发其他监听器的AudioManager.AUDIOFOCUS_LOSS_TRANSIENT。|
|**AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK**|用于申请短暂性音频焦点并要求其它应用降低音量，此时会混音播放，此参数会触发其他监听器的AudioManager.AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK。|
|**AUDIOFOCUS_REQUEST_DELAYED**|用于申请延迟授予的音频焦点：请求成功，但只有在阻止立即授予的条件结束后，才会授予请求者音频焦点。|
|**AUDIOFOCUS_LOSS**|音频焦点丢失。|
|**AUDIOFOCUS_LOSS_TRANSIENT**|暂时失去音频焦点。|
|**AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK**|暂时失去音频焦点，失去音频焦点的程序如果想继续播放，可以降低输出音量，因为新的焦点所有者不要求其他人保持沉默。|
|**AUDIOFOCUS_REQUEST_FAILED**|音频焦点请求失败。|
|**AUDIOFOCUS_REQUEST_GRANTED**|音频焦点请求成功。|

  
  
  
作者：林栩  
链接：[https://www.jianshu.com/p/232dd12e35cb](https://www.jianshu.com/p/232dd12e35cb) 
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。