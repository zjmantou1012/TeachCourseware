![](https://mmbiz.qpic.cn/mmbiz_jpg/jiaXO5U5ibqiaZIiaRECX0TKicP4xibo9ibvG4BrzBHq6TuR2mCOeibkdW2NqjsMIO16GGzRgX0ADT0U2u6icdEmBSJTSNQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

目前市面上大多数IoT模组都支持TCP、UDP、CoAP、LwM2M、MQTT等协议，这里面既有传输层的协议也有应用层的协议，协议众多，适用的场景也不同。但是设计产品时通常只需要运用一种协议，那么怎么来选择一种符合自己产品的应用场景的协议显得尤为重要。本文将介绍TCP、UDP、CoAP、LwM2M、MQTT这5个常用的协议的特点与区别，为设计产品时协议的选择提供参考。  

# 传输层协议TCP与UDP

TCP（传输控制协议，Transport Controll Protocol）、UDP（用户数据报协议，User Data Protocol）同属于传输层协议，为上层用户提供级别的通信可靠性。   ^f63e3d

**传输控制协议（TCP）**：TCP（传输控制协议）定义了两台计算机之间进行可靠的传输而交换的数据和确认信息的格式，以及计算机为了确保数据的正确到达而采取的措施。协议规定了TCP软件怎样识别给定计算机上的多个目的进程如何对分组重复这类差错进行恢复。协议还规定了两台计算机如何初始化一个TCP数据流传输以及如何结束这一传输。TCP最大的特点就是提供的是面向连接、可靠的字节流服务。

**用户数据报协议（UDP）**：UDP（用户数据报协议）是一个简单的面向数据报的传输层协议。提供的是非面向连接的、不可靠的数据流传输。UDP不提供可靠性，也不提供报文到达确认、排序以及流量控制等功能。它只是把应用程序传给IP层的数据报发送出去，但是并不能保证它们能到达目的地。因此报文可能会丢失、重复以及乱序等。但由于UDP在传输数据报前不用在客户和服务器之间建立一个连接，且没有超时重发等机制，故而传输速度很快

## TCP与UDP的区别

|    差异    |   TCP    |    UDP     | 
|:----------:|:--------:|:----------:|
|  是否连接  | 面向连接 | 面向非连接 |
| 传输可靠性 |   可靠   |   不可靠   |
|  应用场合  | 少量数据 |  大量数据  |
|    速度    |    慢    |     快     |

1. TCP面向连接（如打电话要先拨号建立连接）;UDP是无连接的，即发送数据之前不需要建立连接

2. TCP提供可靠的服务。也就是说，通过TCP连接传送的数据，无差错，不丢失，不重复，且按序到达;UDP尽最大努力交付，即不保证可靠交付

3. TCP面向字节流，实际上是TCP把数据看成一连串无结构的字节流;UDP是面向报文的

UDP没有拥塞控制，因此网络出现拥塞不会使源主机的发送速率降低（对实时应用很有用，如IP电话，实时视频会议等）

4. 每一条TCP连接只能是点到点的;UDP支持一对一，一对多，多对一和多对多的交互通信

5. TCP首部开销20字节;UDP的首部开销小，只有8个字节 6、TCP的逻辑通信信道是全双工的可靠信道，UDP则是不可靠信道

## 那么传输层协议是否适合直接运用到物联网设备终端上？

传输层，顾名思义，他只负责传输数据，就好比是一辆运货的货车，但是想让货物完好无损地运到目的地，那就还需要做打包、装车、验货、入库、签回单等工作，这就需要做更多地工作，这些工作也就是应用层协议要做的工作。所以物联网设备终端要想对数据进行稳定、可靠地交互，就需要使用应用层的协议，而不能直接使用传输层的协议。以下将介绍MQTT、CoAP、LwM2M三种适合在物联网设备终端上运用的应用层协议。

# 应用层协议MQTT 与CoAP

## 1、MQTT概述

MQTT(Message Queuing Telemetry Transport，消息队列遥测传输)是IBM开发的一个即时通讯协议，有可能成为物联网的重要组成部分。该协议支持所有平台，几乎可以把所有联网物品和外部连接起来，被用来当做传感器和制动器(比如通过Twitter让房屋联网)的通信协议。

## 2、MQTT协议特点

MQTT协议是为大量计算能力有限，且工作在低带宽、不可靠的网络的远程传感器和控制设备通讯而设计的协议，它具有以下主要的几项特性：

1. 使用发布/订阅消息模式，提供一对多的消息发布，解除应用程序耦合;
2. 对负载内容屏蔽的消息传输;
3. 使用TCP/IP 提供网络连接;
4. 有三种消息发布服务质量：

“至多一次”，消息发布完全依赖底层 TCP/IP 网络。会发生消息丢失或重复。这一级别可用于如下情况，环境传感器数据，丢失一次读记录无所谓，因为不久后还会有第二次发送。“至少一次”，确保消息到达，但消息重复可能会发生。“只有一次”，确保消息到达一次。这一级别可用于如下情况，在计费系统中，消息重复或丢失会导致不正确的结果。小型传输，开销很小(固定长度的头部是 2 字节)，协议交换最小化，以降低网络流量。

## 3、CoAP概述

由于物联网中的很多设备都是资源受限型的，即只有少量的内存空间和有限的计算能力，所以传统的HTTP协议应用在物联网上就显得过于庞大而不适用。IETF的CoRE工作组提出了一种基于REST架构的CoAP协议。CoAP是工作在UDP协议族,采用的是二进制格式，相比起HTTP采用的文本格式，CoAP比HTTP更加紧凑。

## 4、CoAP协议特点

1. 消息模型，以消息为数据通信载体，通过交换网络消息来实现设备间数据通信
2. 对云端设备资源操作都是通过请求与响应机制来完成，类似HTTP，设备端可通过4个请求方法（GET, PUT, POST, DELETE）对服务器端资源进行操作。
3. 协议包轻量级，最小长度仅为4B。
4. 支持可靠传输，数据重传，块传输，确保数据可靠到达。
5. 支持IP多播, 即可以同时向多个设备发送请求
6. 非长连接通信，适用于低功耗物联网场景

## 5、MQTT与CoAP的区别

|类别|MQTT|CoAP|
|:-:|:-:|:-:|
|通信机制|异步|同步|
|连接方式|TCP|UDP|
|通信模式|多对多|多对一|
|使用场景|更适用于推送和IM|物联网|
|功耗|功耗高|功耗低|
|反向控制|可用于反向控制|非长连接，不适合反向控制|

## 那么MQTT和CoAP哪个更适合用于物联网设备上呢？

MQTT和CoAP其实都比较适用于物联网设备上，但是还是要根具实际场景来选择使用。比如设备运行在一个不需要考虑功耗，但是需要实时被控制的场景，例如充电桩、快递柜等场景，则使用MQTT协议比较合适。如果设备通常只有上报数据，且对功耗很敏感的场景，例如水表、燃气表等场景，则使用CoAP协议比较合适。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/jiaXO5U5ibqiab8T0jmJdicAvBO2rLG7bBRdmeibCKQ148Py7ic8G02slvnAIV1hiaGdmZpWy2NcicIYT9oVpYRl0TIiabQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

# LwM2M

随着物联网兴起，万物互联的时代终将到来。但鉴于成本和性能的考虑，设备的资源往往受限，那么就需要一种专门为资源受限的物联网设备设计的协议来满足万物互联的需求，这就是LwM2M协议。

## 1、LwM2M概述：

OMA是一家国际组织，最初定义了一套 OMA-DM的协议，用来远程管理移动终端设备，比如手机开户，版本升级，等等。OMA-DM有着非常广泛的应用，很多运营商比如Verizon Wireless, Sprint都有自己的OMA-DM服务并要求手机/模块入网的时候通过自定义的OMA-DM入网测试。因为物联网的兴起，OMA在传统的OMA-DM协议基础之上，提出了LwM2M协议。2013年底，OMA发布了LwM2M规范。

OMA Lightweight M2M 主要动机是定义一组轻量级的协议适用于各种物联网设备，因为M2M设备通常是资源非常有限的嵌入式终端，无UI,计算能力和网络通信能力都有限。同时也因为物联网终端的巨大数量，节约网络资源变得很重要。

## 2、LwM2M 协议逻辑实体与逻辑接口

### LwM2M 定义了三个逻辑实体:

1. LwM2M Server 服务器

2. LwM2M client 客户端 负责执行服务器的命令和上报执行结果

3. LwM2M 引导服务器 Bootstrap server 负责配置LwM2M客户端.

### 在这三个逻辑实体之间有4个逻辑接口：

Device Discovery and Registration：这个接口让客户端注册到服务器并通知服务器客户端所支持的能力(简单说就是支持哪些资源Resource和对象Object

Bootstrap：Bootstrap server:通过这个接口来配置Clinet - 比如说LwM2M server的URL地址 Device Management and Service Enablement：这个就是最主要的业务接口了。LwM2M Server 发送指令给 Client 并收到回应.

Information Reporting：这个接口是 LwM2M Client 来上报其资源信息的，比如传感器温度。上报方式可以是事件触发，也可以是周期性的。

这三个逻辑实体与四个逻辑接口之间的关系如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/jiaXO5U5ibqiaamVk3dusMOichfVobiayuf9tQFQCvV8ZcMNrwlJicfvO3VyicsbJeKkz9hvwBd4L1OXFWkkAsyUx9SRA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 3、LwM2M 协议栈：

LwM2M 协议栈结构如下图所示：

|                |     |
| -------------- | --- |
| LwM2M Objects  |     |
| LwM2M Protocol |     |
| CoAP           |     |
| DTLS           |     |
| UDP            | SMS |

**LwM2M Objects**: 每个对象对应客户端的某个特定功能实体. LwM2M 规范定义了一下标准Objects，比如

urn:oma:lwm2m:oma:2; (LwM2M Server Object)

urn:oma:lwm2m:oma:3; (LwM2M Access Control Object)

每个object下可以有很多resource. 比如Firmware object可以有Firmware版本号，size等resource.
Vendor可以自己定义object

**LwM2M Protocol**: 定义了一些逻辑操作，比如Read, Write, Execute, Create or Delete.

**CoAP**: 是IETF 定义的Constrained Application Protocol 用来做LwM2M的传输层，下层可以是 UDP 或SMS .UDP 是必须支持的，SMS是可选的。CoAP有自己的消息头，重传机制等。

**DTLS**: 是用来保证客户端和服务器间的安全性的.

## 4、LwM2M与CoAP的关系:

LwM2M的消息没有对称的反馈消息，由于LwM2M承载在CoAP协议上，使用CoAP的get、post、put、delete方式，对于相应消息成功或失败的反馈是通过CoAP协议本身的交互来实现的。LwM2M载荷支持四种格式 plain text、Opaque、TLV、JSON,这四种协议要求服务器端必须都要支持，而在客户端必须支持TLV格式。1

  

猜你喜欢：

[基于LiteOS的智慧农业案例实验分享](http://mp.weixin.qq.com/s?__biz=MzU5MzcyMjI4MA==&mid=2247486988&idx=1&sn=ca0e4a5d9c1ce7fd44c81c9d7a05178f&chksm=fe0d60cbc97ae9dd81271b3279007b5cfdf81dd39b3212bd58aacfe69897a470a3472afff584&scene=21#wechat_redirect)

[笔记：编写简单的内核模块](http://mp.weixin.qq.com/s?__biz=MzU5MzcyMjI4MA==&mid=2247487110&idx=1&sn=26a9153ad20e3de91a1a0e3df24046db&chksm=fe0d6041c97ae957e0c35e6096919ffbfc62679571a1ea1ed16e9a10077d3a44fd8c761b30b4&scene=21#wechat_redirect)  

[【Linux笔记】设备树实例分析](http://mp.weixin.qq.com/s?__biz=MzU5MzcyMjI4MA==&mid=2247487108&idx=1&sn=f86b192308a3f08ede10b38a64fd9c52&chksm=fe0d6043c97ae95558fda890dc91c8b9f2e28274d938804264e0e4defcd7b16d39f185c01b38&scene=21#wechat_redirect)  

[【Linux笔记】通俗易懂的Linux驱动基础](http://mp.weixin.qq.com/s?__biz=MzU5MzcyMjI4MA==&mid=2247486771&idx=1&sn=0df4aa4c3673b5a25bfa0b9e09122f97&chksm=fe0d63f4c97aeae26af717e99050aa7bbb183d3ee9aac76e27222df8804f26d6519bb90c07e2&scene=21#wechat_redirect)  

[【Linux笔记】pc机_开发板_ubuntu互ping实验](http://mp.weixin.qq.com/s?__biz=MzU5MzcyMjI4MA==&mid=2247486700&idx=1&sn=601a9d5d46796841a2c05f76c607f42b&chksm=fe0d622bc97aeb3d007e316c4507be9057171fccbcd13ad6f30e06dafd0c899c6627c642c0f3&scene=21#wechat_redirect)  
[【Linux笔记】挂载网络文件系统](http://mp.weixin.qq.com/s?__biz=MzU5MzcyMjI4MA==&mid=2247486719&idx=1&sn=0b3cb8a3b395c8a934d6f9a7dee620d2&chksm=fe0d6238c97aeb2e326cb9635689979d2e106f78d92d85369a8128ed1836c8c1834b7b3ef590&scene=21#wechat_redirect)

[学习STM32的一些经验分享](http://mp.weixin.qq.com/s?__biz=MzU5MzcyMjI4MA==&mid=2247486788&idx=1&sn=ff32c5cecb481bb3553fb4feb5806af5&chksm=fe0d6383c97aea95ac1e494cd77a9066b5909bf207f025a2424cd1f8fd9923adf713f87a4e1c&scene=21#wechat_redirect)

[从单片机工程师的角度看嵌入式Linux](http://mp.weixin.qq.com/s?__biz=MzU5MzcyMjI4MA==&mid=2247486747&idx=1&sn=9f03990d5042a660959551d35af50d58&chksm=fe0d63dcc97aeaca16857cbef9e782ef697269986da9215c244f18961d9748b13b105b9ca45e&scene=21#wechat_redirect)

  

后台回复：加群。添加ZhengN微信，加入【嵌入式大杂烩】交流群
![](https://mmbiz.qpic.cn/mmbiz_png/PnO7BjBKUz81oloND0c1h26ficicA84t1wicPE6GBMXDiaUjLOVGT7tx9qPSica5v52UvXUQicmrOMibWNZbISemU29yQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)