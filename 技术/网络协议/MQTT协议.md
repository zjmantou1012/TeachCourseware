---
author: zjmantou
title: MQTT协议
time: 2023-09-03 周日
tags:
  - 技术
  - Iot
  - 网络协议
  - 通信协议
  - MQTT
---


MQTT全称Message Queuing Telemetry Transport，直译过来是消息队列遥测传输的意思

百度百科对于MQTT协议的定义如下：

MQTT(消息队列遥测传输)是ISO 标准(ISO/IEC PRF 20922)下基于发布/订阅范式的消息协议。它工作在 [TCP/IP协议族](https://link.juejin.cn?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2FTCP%252FIP%25E5%258D%258F%25E8%25AE%25AE%25E6%2597%258F "https://baike.baidu.com/item/TCP%2FIP%E5%8D%8F%E8%AE%AE%E6%97%8F")上，是为硬件性能低下的远程设备以及网络状况糟糕的情况下而设计的发布/订阅型消息协议，为此，它需要一个[消息中间件](https://link.juejin.cn?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E6%25B6%2588%25E6%2581%25AF%25E4%25B8%25AD%25E9%2597%25B4%25E4%25BB%25B6 "https://baike.baidu.com/item/%E6%B6%88%E6%81%AF%E4%B8%AD%E9%97%B4%E4%BB%B6") 。
![MQTT工作原理](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031425430.png)

#### **优点**：	
- **空间上去耦合**：信息发布者和信息接收者不需要建立直接联系，不需要知道对方的IP地址，端口等信息	
- **时间上去耦合**：发布者和接收者不需要同时在线。

#### **缺点**：	
- 发布者、接收者必须订阅相同的主题，	
- 发布者并不确定接收者是否接受到了推送2、客户端和服务器的角色和职责

#### 客户端服务端职责
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031430423.png)

# 主题
![mqtt主题](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031432050.png)

主题通配符：（订阅消息时使用）
单级通配符 +：+可以匹配单级，表示该级可以是任意主题不受限定；
多级通配符 # : 可以匹配多级，通常出现在主题末尾，表示某一类主题下的所有子主题
系统保留topic $：以$开头的是服务器保留的主题
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031435453.png)

# 消息质量QoS

![QoS](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031437906.png)

1. 至多一次：完全依赖TCP/IP网络，会发生丢失和重复，应用场景如：收集传感器数据，中间丢失了无所谓；
2. 至少一次：确保消息到达，但可能会重复收到消息，
3. 只有一次：应用场景：计费系统

# 数据格式
## 消息组成
一个消息由三个部分组成：固定头、可变头、负载
### 固定头
共 **2** 个字节；
- 第一个字节高4位表示消息类型；
- 第一个字节低4位根据消息类型定义不同；
- 第二个字节声明可变头和负载的长度；

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031453925.png)

### 可变头

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031453114.png)

不同消息类型有不同的格式

### 负载（Payload）

根据消息类型不同变化


## 建立连接过程

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031455647.png)

### CONNECT报文
1. ClientID：每个客户端的唯一ID，如果有重复则连接不成功
2. Username，password：由服务器分配的用户名和密码，
3. CleanSession（持久化会话）：客户端断线重连时，服务器需不需要将掉线期间的消息保留并重发
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031459298.png)

如果CleanSeesion为false，服务端会发送缓存的数据到掉线重连的设备，否则就会丢弃并建立新的会话连接。

#### willRetain 遗愿
为了让订阅者感知到发布者掉线。  
	发布者在连接时就在服务器发布一条消息，服务器会存储这条遗愿，不会立即发布给订阅者，如果发布者自己掉线了，就会把发布者的遗愿推送给订阅者。

#### KeepAlive
心跳，保持长连接  
当客户端没有消息发送给服务端时，会发送心跳连接

#### 例子
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031512178.png)

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031512535.png)

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031513417.png)

## 消息发布（PUBLISH）
包括客户端向服务端PUBLISH 和 服务端向客户端PUBLISH  
### PUBLISH内容
组成：
- **Packet ID**：数据包名，每一条消息都有一个唯一的数据包名（QoS0没有此ID）
- **TopicName**：主题名称，主题信息  
- **QoS**：信息传递质量等级  
- **RetainFlag**：保留消息，会保存在服务端，新的设备连接上来之后服务端会发送这一条消息给到新的设备。能保证新连接的设备立刻得到一条消息。
- **Payload**：
- **DupFlag**：
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031527631.png)

### 报文格式
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031530513.png)

### PUBACK数据包
只有在QoS1、QoS2质量下才会有puback响应包。

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031531480.png)

## 订阅消息

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031532241.png)

客户端通过SUBSCRIBE消息包向服务器订阅若干主题（带传输质量），服务器向客户端发送SUBACK消息包返回订阅情况。
同时客户端也可以向服务器发送UNSUBSCRIBE消息包来取消订阅主题，而服务器也会发送UNSUBACK消息包并将其请求的结果返回给客户端  
消息的订阅与取消不管主题的QoS等级，均会带有packetID。 

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031533720.png)

### SUBSCRIBE消息报文格式
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031533754.png)

### SUBACK报文格式

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031533317.png)

### UNSUBSCRIBE报文格式

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031534616.png)

### UNSUBACK报文格式

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031534040.png)

# MQTT开源框架

1. MQTT服务器构建的开源框架：Mosquitto

官方网站:[mosquitto.org/](https://link.juejin.cn?target=http%3A%2F%2Fmosquitto.org%2F "http://mosquitto.org/")

GitHub开源页面：[github.com/eclipse/mos…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Feclipse%2Fmosquitto "https://github.com/eclipse/mosquitto")

2. MQTT客户端设计软件：paho

[www.eclipse.org/paho/](https://link.juejin.cn?target=http%3A%2F%2Fwww.eclipse.org%2Fpaho%2F "http://www.eclipse.org/paho/")

GitHub开源页面：

[github.com/eclipse/pah…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Feclipse%2Fpaho.mqtt.python "https://github.com/eclipse/paho.mqtt.python")（python）

[github.com/eclipse/pah…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Feclipse%2Fpaho.mqtt.android "https://github.com/eclipse/paho.mqtt.android") (安卓)

[github.com/eclipse/pah…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Feclipse%2Fpaho.mqtt.java "https://github.com/eclipse/paho.mqtt.java") （Java）

[github.com/eclipse/pah…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Feclipse%2Fpaho.mqtt.javascript%25EF%25BC%2588JavaScript "https://github.com/eclipse/paho.mqtt.javascript%EF%BC%88JavaScript")）

3. MQTT客户端模拟： MQTT.fx

[mqttfx.jensd.de/](https://link.juejin.cn?target=http%3A%2F%2Fmqttfx.jensd.de%2F "http://mqttfx.jensd.de/")

# MQTT的安全
MQTT发送数据时是明文发送的，而且还包含了clientID，username和password，所以应该对其进行加密。

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309031535560.png)



  

作者：熊爸天下  
链接：https://juejin.cn/post/6844904078426767368  
来源：稀土掘金  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



作者：熊爸天下  
链接：https://juejin.cn/post/6844904078426767368  
来源：稀土掘金  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。