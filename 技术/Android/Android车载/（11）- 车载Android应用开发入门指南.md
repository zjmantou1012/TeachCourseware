---
author: zjmantou
title: （11）- 车载Android应用开发入门指南
time: 2023-09-18 周一
tags:
  - Android
  - 车联网
---
### 1. 前言 - 移动互联网退潮下的汽车大战

将时间回退到2017年我大学刚毕业时，彼时移动互联网就已经开始退潮，各大个培训机构也纷纷停止了Android相关的培训，曾经热火朝天的应用开发从那时起，就开始走向下坡路，小程序以及众多跨平台框架也让市场对Android原生开发的需求逐年降低，市场需求的降低也造就了Android开发的面试变得史无前例的“卷”。

终于我在2019年选择离开了互联网，投身当时还不是非常火热的车载Android领域继续从事Android原生开发。而这一年中国首个外商独资的整车制造项目，“上海特斯拉超级工厂”开工了。

特斯拉在智能化和电子化上的巨大优势将智能汽车推向了一个全新的高度，先进的自动驾驶以及BMS电池管理系统，深深震撼了全世界的人，在当时的国人眼中特斯拉几乎就是新能源汽车的代名词，时至今日，Model Y和Model 3已也依然是新能源汽车领域的畅销车型。

众所周知汽车工业是发达国家重要的经济支柱，而中国是世界上最大汽车生产和销售国，特斯拉的热销立马就引发了一场 **鲶鱼效应** ，国内外的汽车制造商纷纷开始布局智能化汽车，汽车工业走向了软件定义汽车的时代。软件定义汽车的核心思想是，决定未来汽车的是以人工智能为核心的软件技术，车载软件在汽车领域的重要性首次被拔高到了前所未有的高度，就这样一场轰轰烈烈的车载软件技术大战上演了。

### 2. 智能汽车座舱基本结构

在从事车载Android应用开发前，必须要对汽车座舱的基本结构有一个大体的认知，只有意识到汽车座舱是一种与手机完全不同的架构，才能更好的助力我们日后学习车载Android应用的开发。下面就来介绍一个比较主流的车载操作系统架构。

注意：并不是所有的车载操作系统都采用了下面的架构，比如，特斯拉采用的是基于Linux一套架构。

![](https://cdn.nlark.com/yuque/0/2022/png/26044650/1670762581867-e1b8d7c2-69b9-4865-acaa-1c3b7a497c42.png)

上面就是目前主流汽车座舱采用技术架构，我们从上到下依次介绍：

#### T-BOX

T-Box又称TCU（车联网控制单元），指安装在汽车上用于控制跟踪汽车的嵌入式系统，是车载信息交互系统核心部件，有了它汽车才能实现联网功能，所以也起到中央网关的作用。通常包括GPS单元、移动通讯外部接口电子处理单元、微控制器、移动通讯单元以及存储器等。  
对车辆，T-Box可提供车辆故障监控、电源管理、远程升级、数据采集、智慧交通等功能，对车主，T-Box可为提供车辆远程控制、安防服务等功能。  
T-BOX属于外围硬件，与中控、仪表并不集成在一个主板上。

#### SOC

SoC的定义多种多样，由于其内涵丰富、应用范围广，很难给出准确定义。一般说来， SoC称为系统级芯片，也有称片上系统（System on Chip），意指它是一个产品，是一个有专用目标的集成电路，其中包含完整系统并有嵌入软件的全部内容。  
车载Soc和我们最常见的手机Soc非常类似，内部集成了CPU和GPU。目前最主流的车载Soc是高通的SA8155，它就是高通在手机Soc骁龙855的基础上发展而来的。

#### MCU

微控制单元(Microcontroller Unit；MCU) ，又称单片微型计算机(Single Chip Microcomputer )或者单片机，是把中央处理器(Central Process Unit；CPU)的频率与规格做适当缩减，并将内存(memory)、计数器(Timer)、USB、A/D转换、UART、PLC、DMA等周边接口，甚至LCD驱动电路都整合在单一芯片上，形成芯片级的计算机。

一般汽车座舱内，集成SOC的主板上也会额外集成一个或多个MCU。

#### AutoSAR

Adaptive AutoSAR 是一种适用于高级自动驾驶的软件架构平台，提要提供高性能的计算和通信，提供灵活的软件配置，支撑应用的更新。  
Adaptive AutoSAR 的主要架构分为硬件层、ARA（AutoSAR Run-timeFor Adaptive实时运行环境）以及应用层。  
应用层包含的应用程序模块（AA）运行在ARA之上，每个AA以独立的进程运行。ARA由功能集群提供的应用接口组成，他们属于自适应平台。自适应平台提供Adaptive AutoSAR 的基本功能和标准服务。  
每个AA可以向其他AA发生服务。基于这种架构，整车的功能之间可以解耦。

#### Hypervisor

一种运行在基础物理服务器和操作系统之间的中间软件层，可允许多个操作系统和应用共享硬件。也可叫做VMM（ virtual machine monitor ），即虚拟机监视器。  
目前主流的汽车座舱，都是同时在一个Soc上运行着两个不同的操作系统，一个是显示汽车仪表盘的QNX系统，另一个用于车载信息娱乐的Android系统，其底层技术原理就是Hypervisor。

#### QNX

QNX是一种商用的、遵从POSIX规范的类Unix实时操作系统，目标市场主要是面向嵌入式系统，具备高运行效率、高可靠性特点，并在工控领域拥有近40年的使用经验，被广泛应用于汽车、轨道交通、航空航天等对安全性、实时性要求较高的领域。  
QNX在车载操作系统市场的占有率超过75%，在更注重生态和内容的车载娱乐系统占有率也超过60%，而在强调安全性的仪表盘以及驾驶辅助领域，QNX的市占率更是达到了近100%。

2010年QNX被加拿大RIM公司收购，而这家公司就是黑莓BlackBerry的母公司。

#### SOA

SOA（Service-OrientedArchitecture）是一种基于业务实现的粗粒度松耦合的面向服务的分布式架构，即实现业务和技术的分离，又实现业务和技术的自由组合。  
以位置服务为例，很多车内应用会用到位置信息，像天气、拍照、导航，这些应用根据自身服务有不同的需求，对位置信息的处理各不相同，SOA就可以很好地解决这个问题。  
SOA原本是服务器开发中用到的技术，现如今也被用在车载操作系统领域，但是目前关于SOA的技术规范比较混乱，国内主机厂商外对于SOA的实现方式也有区别。  
SOA并不车载操作系统必须的，其实目前为止已经上市的车型中，很少采用了SOA架构，所以它还只是车载操作系统未来的一个发展方向。

2021年上汽零束率先发布业界首个车载SOA软件架构规范。威马汽车科技集团旗下的W6号称国内首款采用SOA的量产车。

#### 车载以太网

车载以太网是一种用以太网连接车内电子单元的新型局域网技术，与传统以太网使用4对非屏蔽双绞线电缆不同，车载以太网在单对非屏蔽双绞线上可实现100Mbit/s，甚至1Gbit/s的传输速率，同时还满足汽车行业对高可靠性、低电磁辐射、低功耗、带宽分配、低延迟以及同步实时性等方面的要求。

车载以太网的设计是为了满足车载环境中的一些特殊需求。例如：满足车载设备对于电气特性的要求（EMI/RF）；满足车载设备对高带宽、低延迟以及音视频同步等应用的要求；满足车载系统对网络管理的需求等。因此可以理解为，车载以太网在民用以太网协议的基础上，改变了物理接口的电气特性，并结合车载网络需求专门定制了一些新标准。针对车载以太网标准，IEEE组织也对IEEE 802.1和IEEE 802.3标准进行了相应的补充和修订。

#### CAN

CAN是控制器域网 (Controller Area Network, CAN) 的简称，是由研发和生产汽车电子产品著称的德国BOSCH公司开发了的，并最终成为国际标准（ISO11898）。是国际上应用最广泛的现场总线之一。 在北美和西欧，CAN总线协议已经成为汽车计算机控制系统和嵌入式工业控制局域网的标准总线，并且拥有以CAN为底层协议专为大型货车和重工机械车辆设计的J1939协议。近年来，其所具有的高可靠性和良好的错误检测能力受到重视，被广泛应用于汽车计算机控制系统和环境温度恶劣、电磁辐射强和振动大的工业环境。  
CAN在车载操作系统&应用开发中使用非常广泛，车载Android的核心服务之一 - CarService本质上就是将外部硬件通信报文解析成上层应用可以识别的数据，这里的通信报文目前普遍都是CAN报文。

CAN通信在车载中使用的是如此广泛，以至于作为Android程序员，我们都不得不去学习CAN仿真测试工具的使用，有时候甚至需要我们去阅读、解析CAN报文。  
值得一提的是CAN仿真测试工具非常昂贵，虽有国产替代，但目前依然普遍采用德国维克多公司出品的各类工具和软件，价格在数万元到数十万元不等。

#### 3D HMI设计工具 & 嵌入式图形引擎

随着车载Soc算力的提高，现代座舱越来越喜欢引入3D化的图形界面，3D化的界面可以实时生成动画反馈，大大提升了界面的美观性和易用性。目前车载开发中主流的3D HMI设计工具&图形引擎有老牌的游戏开发工具如Unity 3d、Unreal（虚幻），也有专用于汽车HMI设计&图形显示的 — Kanzi 。

2016年芬兰汽车软件公司Rightware以及旗下产品Kanzi，被国内的汽车软件供应商中科创达收购。

上面介绍了汽车座舱的基础知识，Android应用程序员说到底还是负责在座舱中控，编写各类型的应用，下面就来介绍车载应用与互联网应用的不同之处。

### 3. 车载应用开发

车载Android应用说到底就是，在车载Android系统中嵌入一系列系统级应用，这里既包含与用户存在交互的HMI应用，也包含在后台运行没有HMI的Service应用。

一般而言，车载应用复杂度比一般的互联网应用还要低一些。

常见有HMI的车载应用如，车载空调、多媒体应用、桌面、SystemUI、系统设置、车控车设、蓝牙电话以及一些第三方应用等等。

没有HMI的应用有，CarService、AudioService、AccountService等等。在车载应用开发中需要定制大量的Service，这也是应用开发中工作量比较大的一部分。

#### 3.1 系统级应用与普通应用的区别

系统应用需要嵌入到Android ROM中运行，虽然普通的应用也可以嵌入到ROM中，但是系统应用可以调用Android SDK的内部API，而这一点是普通应用做不到的，总得来说系统应用具有以下特点

- 可以访问Android SDK内部的API
- 不需要申请动态权限
- 可配置开机自启动
- 必须对应用进行签名

接下来我们实际上手编写一个系统级应用。

#### 3.2 编写一个系统级应用

编写Android系统应用与普通的Android应用基本相同，我们首先在AndroidStudio中编写一个demo，只需要一个空白的Activity和Application即可。

```Java
public class DemoApp extends Application {

    private Handler handler;

    @Override
    public void onCreate() {
        super.onCreate();
        Log.e("TAG", "onCreate: start");
        handler = new Handler(Looper.getMainLooper());
        handler.postDelayed(new Runnable() {
            @Override
            public void run() {
                showView();
            }
        },5000);
    }

    private void showView(){
        WindowManager manager = getSystemService(WindowManager.class);
        View view = new View(this);
        WindowManager.LayoutParams params = new WindowManager.LayoutParams(WindowManager.LayoutParams.MATCH_PARENT,WindowManager.LayoutParams.MATCH_PARENT);
        params.type = WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY;
        manager.addView(view,params);
    }
}
```

上面的application逻辑很简单，app启动5秒后，弹出一个全屏的Window的。

接下来在AndroidManifest.xml中注册application。

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
  package="com.example.car"
  android:sharedUserId="android.uid.system">

  <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
  <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />

  <application
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:persistent="true"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:supportsRtl="true"
    android:theme="@style/Theme.First">

    <activity
      android:name=".MainActivity"
      android:exported="true">
      <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
      </intent-filter>
    </activity>
  </application>

</manifest>
```

在上面源码中我们需要关注两个普通应用用不到的属性：  
**android:sharedUserId**  
将与其他应用程序共享的 Linux 用户 ID 的名称。默认情况下，Android 会为每个应用分配自己唯一的用户 ID。但是，如果为两个或多个应用将此属性设置为相同的值，则它们将共享相同的 ID，前提是它们的证书集相同。具有相同用户 ID 的应用可以访问彼此的数据，如果需要，可以在同一进程中运行。  
开发系统应用时，此项不是必须配置的。配置为android.uid.system后，该应用会变成system用户，可以访问一些system用户才能访问的空间。

**android:persistent**  
配置应用程序是否应始终保持运行，默认为false。设为true之后，应用在开机广播发出之前就会自行启动，而且应用被杀死后，也会立即重启。  
开发系统应用时，此项不是必须配置的。

#### 3.3 测试系统应用

##### 3.3.1 准备测试环境

测试系统应用就比较麻烦了，由于手边没有开发板，只能基于模拟器进行测试，所以就必须下载Android的源码，并使用Android源码环境编译出带有系统签名的APK。  
下载、编译Android源码 请参考 ：[Android车载应用开发与分析（1） - Android Automotive概述与编译](https://www.jianshu.com/p/bbc02e0f6575)  
完成Android源码编译后，我们将编写好的FirstCarApp部分源码拷贝到 /aosp/packages/apps/Car/ 下，  
基于Android源码环境的app工程结构与基于Gradle的AndroidStudio工程结构是完全不一样的，目录结构如下：

![](https://cdn.nlark.com/yuque/0/2022/png/26044650/1670762652841-e77d53a5-097e-4715-82a2-905565020b2f.png)

你应该注意到了 src 目录下没有androids studio工程结构中的main/java  
需要强调的是，这种基于原生的写法，并不常用。实际开发中，我们依然是在Android Studio中开发完毕，将源码提交到gerrit上，后续的编译、签名、复制的过程会有jenkins帮我们完成。

  
  
作者：林栩  
链接：[https://www.jianshu.com/p/ecb5b2da73de](https://www.jianshu.com/p/ecb5b2da73de)
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。