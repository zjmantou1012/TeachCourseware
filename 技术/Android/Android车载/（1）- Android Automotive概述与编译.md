---
author: zjmantou
title: （1）Android Automotive概述与编译
time: 2023-09-18 周一
tags:
  - Android
  - 车联网
  - Automotive
---
## 1. Android开发者的新赛道

在智能手机行业初兴起时，包括BAT在内许多传统互联网企业都曾布局手机产业，但是随着手机市场的基本定型，造车似乎又成了各大资本下一个追逐的方向。百度、小米先后宣布造车，阿里巴巴则与上汽集团共同投资创立了，面向汽车全行业提供智能汽车操作系统和智能网联汽车整体解决方案的斑马网络，一时间造车俨然成了资本市场的下一个风口。

而作为移动端操作系统的霸主 - **Android**，也以一种新的姿态高调侵入造车领域，这就是 Android 车载信息娱乐系统 - **Android Automotive。**

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670157266527-de641958-e89f-439e-a9e5-357eb4b0bb6d.webp)

## 2. 什么是Android Automotive？

Android Automotive 是一个基本 Android 平台车载信息娱乐系统，简称IVI（In-Vehicle Infotainment）。

Android Automotive系统赋予了车厂在`IVI` 系统中预装 Android 应用的能力，而大量的Android开发从业者，也降低的`IVI`系统以及应用的开发成本

### 2.1 Android Automotive 和 Android

- **Android Automotive 就是 Android 平台**。Android Automotive 并非 Android 的分支或并行开发版本。它与手机和平板电脑等设备上搭载的 Android 使用相同的代码库，位于同一个存储区中。它基于开发时间逾 10 载的强大平台和功能集构建而成，因此能够利用现有的安全模型、兼容性计划、开发者工具和基础架构，同时继续保持较高的可定制性和可移植性，完全免费提供并且开源。
- **Android Automotive 扩展了 Android 平台**。在将 Android 打造为功能完善的信息娱乐平台的过程中，我们增加了对汽车特定要求、功能和技术的支持。Android Automotive 将是一个一站式全栈车载信息娱乐平台，就像现在的 Android 系统之于移动设备一样。

### 2.2 Android Automotive 和 Android Auto

- **Android Auto** 是一个基于用户的手机运行的平台，可通过 USB 连接将 Android Auto 用户体验投射到兼容的车载信息娱乐系统。Android Auto 支持专为车载用途而设计的应用。如需了解详情，请访问 [developer.android.com/auto](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.com%2Fauto%2Findex.html)。
- **Android Automotive** 是直接基于车载硬件运行的操作系统和平台。它是一个可定制程度非常高的全栈开源平台，可为信息娱乐体验提供强大的技术支持。Android Automotive 支持专为 Android 打造的应用，以及专为 Android Auto 打造的应用。

### 2.3 Android Automotive 的架构设计概述

Android Automotive作为车载信息娱乐系统必须具备查看、控制整车其它子系统（如 空调）的能力，但是不同的制造商提供的总线类型和协议之间有很大差异，例如控制器局域网 (CAN) 总线、区域互连网路 (LIN) 总线、面向媒体的系统传输 (MOST) 总线以及汽车级以太网和 TCP/IP 网络（如 BroadR-Reach）。

Android Automotive 的硬件抽象层 (HAL) 为 Android 框架提供了一致的接口（无需考虑物理传输层），系统集成商可以将特定功能的平台 HAL 接口（如 空调）与特定于技术的网络接口（如 CAN 总线）连接，以实现车载 HAL 模块。

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670157306569-0e01fa4b-228a-4790-b053-f4b613891892.webp)

image

- **Car API**：内有包含 `CarSensorManager` 在内的 API。如需详细了解受支持的 API，请参阅`/platform/packages/services/Car/car-lib`。
- **CarService**：位于 `/platform/packages/services/Car/`。
- **车载 HAL**：用于定义 OEM 可以实现的车辆属性的接口。包含属性元数据（例如，车辆属性是否为 int 以及允许使用哪些更改模式）。位于 `hardware/libhardware/include/hardware/vehicle.h`。如需了解基本参考实现，请参阅 `hardware/libhardware/modules/vehicle/`。

作为车载应用开发者，对于Android Automotive 的架构，有个基础认知即可并不影响我们后续对车载应用开发的学习。

## 3. 创建Android Automtive模拟器

为了让便于我们对Android Automotive有一个直观上的认知，我们可以先在Android Studio上创建一个模拟器。下面的Android Automtive模拟器创建步骤基于MAC OS版Android Studio Arctic Fox

- 1.在Preferences(Windows下是Settings) -> Appearance&Behavior -> System Settings ->Updates 中将检查更新的channel改为`Canary Channel`

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670157349260-73fc379d-ea9e-4d01-82c1-863a1ed00d20.webp)

image

- 2.在创建模拟器的时候选择一个你需要的 Android Automotive 镜像

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670157356243-ac67402d-e8ec-4a26-943e-0f691459b7a0.webp)

image

- 3.最后，我们就可以使用Android Automotive的模拟器了

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670157365227-77c6b74a-f7f3-4f1d-9a1b-4068bade58e7.webp)

image

模拟器到此为止就创建完毕了，可以随便把玩一波，看看google是如何理解车载娱乐系统的。

不得不说的是，在国内实际的车载应用开发中，我们很少会把应用直接跑在模拟器上，其中一个原因就是AS创建的Android Automotive模拟器是production版本，我们并不能获取root、remount权限，这非常不利于我们的调试。

这里额外提一句，通过Android Studio创建的手机模拟器，无需任何操作就可以获取root权限。然后还可以通过控制台在`Android/sdk/emulator`目录下，运行下面的指令来开放`remount`权限

```bash

emulator -writable-system -netdelay none -netspeed full -avd 模拟器的名字 

```

为了在模拟器中获取root、remount权限，以及方便我们之后研究Android Automotive上原生应用的原理，这里我们接着来介绍一下如何下载 Android Automotive 源码，以及如何编译源码。

## 4. 下载&编译 Android Automotive

由于众所周知的原因国内下载AOSP速度非常缓慢，所以以下步骤使用清华大学的AOSP镜像。下载以及编译环境推荐使用Ubuntu系统，编译Android 9及以上的AOSP，硬盘需要预留500GB以上的空间，内存也至少需要8GB以上。以下内容基于如下环境编写。

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670157420977-a85855da-4f9a-489a-aefe-a28ef5f17a6e.webp)

image

#### 1. 下载repo工具

```bash
mkdir ~/bin 
PATH=~/bin:$PATH 
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo 
chmod a+x ~/bin/repo 
```

#### 2. 下载初始化包

从清华大学开源镜像站下载初始化包。由于首次同步需要下载约 130GB 数据，过程中任何网络故障都可能造成同步失败，强烈建议直接使用初始化包进行初始化。使用方法如下:

```bash
wget -c https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar # 下载初始化包，可以用下载工具代替 
tar xf aosp-latest.tar #解压初始化包 
cd aosp   # 解压得到的 AOSP 工程目录 
# 这时 ls 的话什么也看不到，因为只有一个隐藏的 .repo 目录 
```

此后，每次只需运行 `repo sync` 即可保持与主分支同步。当然我们也可以选择我们指定的Android版本，继续如下的操作

```bash
cd .repo/manifests 
git branch -a # 查看Android分支 
```

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670157515853-837ac368-6c28-475e-81b1-847f2a338c9e.webp)

```bash
repo init -b android-11.0.0.0_r40 # 切换到Android 11 
repo sync # 再同步一遍即可得到基于Android 11的完整目录 
```

#### 3. 准备编译环境

在Ubuntu的控制台中执行下列指令来安装编译AOSP所必需各类型工具

```bash
sudo apt-get update 
sudo add-apt-repository ppa:openjdk-r/ppa 
sudo apt-get update 
sudo apt-get install openjdk-8-jdk 
sudo apt-get install git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev libgl1-mesa-dev libxml2-utils xsltproc unzip 
sudo apt-get install -y lib32stdc++6  
sudo apt-get install git 
sudo apt-get install libssl-dev 
sudo apt-get install libncurses5 
```

#### 4. 开始编译

- 1.在aosp根目录的控制台中执行下列指令，初始化脚本

```bash
source build/envsetup.sh 
```

- 2.使用**lunch**选择编译的目标类型。因为是在电脑上调试编译出的版本，所以这里我们选择 aosp_car_x86_64-userdebug或aosp_car_x86-userdebug。

```bash
lunch # 打开选择菜单 
11 # 选择 aosp_car_x86_64-userdebug 
```

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670157647121-110f6bf1-d42f-4b7a-8a97-f5662963a4ee.webp)

- 3.使用`make -jX`编译源码。电脑的CPU核心数越多，X可以设定的值越大，编译速度也就越快，一般可以直接设为cpu核心数，如果你的CPU支持超线程还可以再乘以2。

```bash
make -j8 # 开始编译 
```

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670157669770-4a9012fb-fbae-4fe8-9b5c-41d186337df3.webp)

image

编译时间取决于你电脑的性能，在机械硬盘下首次编译约耗时5-7个小时。控制台中提示Successful，即表示编译成功。

- 4.启动模拟器

```bash
emulator -partition-size 1500  
```

漫长的开机动画之后，模拟器顺利启动。可以看出我们自行编译的模拟器，launcher 界面以及预装的APP与Android Studio中提供的 Android Automotive 还是有很大区别的。在之后的时间里面，我们就来一一解析的这些系统应用的运行原理。

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670157719909-611f90f5-4125-4342-9afe-f568cef80ee9.webp)

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670157728592-6dfd76a8-231c-4fb7-abb9-58b6504855ad.webp)

#### 5.常见错误

##### 1.各类编译环境报错

一般环境报错，百度一下基本上都解决。在这里强烈建议在 Ubuntu 16 或以上的Linux环境下编译Android的源码！我个人尝试过在 Mac OS 和Windows OS下编译Android源码，各种错误层出不穷，而换到 Ubuntu 环境下这些错误几乎就都没有了。

##### 2. This user doesn't have permissions to use KVM

解决方案，在控制台执行以下指令

```bash
sudo chown 用户名 -R /dev/kvm  
```

##### 3. warning: repo is not tracking a remote branch, so it will not receive updates. repo reset: error: Entry 'xxxxx.py' not uptodate. Cannot merge.fatal: Could not reset index file to revision 'v2.15.4^0'

解决方案:

```bash
cd .repo 
cd repo 
ls  
```

在控制台确认一下报错的xxx.py在不在这个文件下，如果在不，需要去别文件下看一下。一般报错的xxx.py就是目录下的。

```bash
git log # 找到倒数第二个conmmit-id  
```

![](https://cdn.nlark.com/yuque/0/2022/png/26044650/1670157791939-7fa83610-339f-44a7-bb98-a273099df53a.png)

![](https://cdn.nlark.com/yuque/0/2022/png/26044650/1670157797087-36fed663-fb86-4813-a58e-b0ac74b283f8.png)

```bash
git reset --hard 5637afcc60fdbd38fc0790ea84d5dcb901ec5959 
git pull ## 重新拉取 
```

同步完毕后再执行repo sync.就可以了

## 5.参考资料

[Automotive | Android 开源项目 | Android Open Source Project (google.cn)](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fdevices%2Fautomotive)

[AOSP | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://links.jianshu.com/go?to=https%3A%2F%2Fmirrors.tuna.tsinghua.edu.cn%2Fhelp%2FAOSP%2F)

  
  
作者：林栩  
链接：[https://www.jianshu.com/p/bbc02e0f6575](https://www.jianshu.com/p/bbc02e0f6575)
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。