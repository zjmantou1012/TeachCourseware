---

author: zjmantou
title: 物联网Iot
time: 2023-09-02 周六
tags:
- 笔记
- Iot
- 网络协议
- 通信协议

---



# 网络协议
物联网在传输上，更需要稳定、可靠的协议，就需要应用层协议，而不能直接使用传输层的协议，常见的有MQTT、CoAP、LwM2M
## MQTT
即Message Queuing Telemetry Transport，消息队列遥测传输。
### 特点
为大量计算能力有限，且工作在低带宽、不可靠的网络的远程传感器和控制设备通讯而设计的协议：
1. 使用发布/订阅模式，一对多，低耦合；
2. 对负载内容屏蔽的消息传输；
3. 使用TCP/IP提供网络链接；
4. 三种消息发布质量（Qos）
	1. 至多一次：完全依赖TCP/IP网络，会发生丢失和重复，应用场景如：收集传感器数据，中间丢失了无所谓；
	2. 至少一次：确保消息到达，但可能会重复收到消息，
	3. 只有一次：应用场景：计费系统
5. 开销小（固定长度的头部字节为2字节）

## CoAP
基于REST架构，工作在UDP协议上，采用二进制格式，相比HTTP协议的文本格式更紧凑，能够应对很多物联网设备受限的内存空间和计算能力；

### 特点
1. 消息模型，以消息为数据通信载体，通过交换网络消息来实现设备间数据通信；
2. 类似HTTP，通过请求/响应机制来完成（GET、PUT、POST、DELETE）
3. 轻量级协议包，最小长度4B；
4. 支持可靠传输、数据重传、块传输，确保数据可靠到达。
5. 支持IP多播，同时向多个设备发送请求；
6. 非长链接，适合低功耗的物联网场景；  

## MQTT与CoAP比较

| 类别     | MQTT             | CoAp                     |
| -------- | ---------------- | ------------------------ |
| 通讯机制 | 异步             | 同步                     |
| 连接方式 | TCP              | UDP                      |
| 通讯模式 | 多对多           | 多对一                   |
| 使用场景 | 更适用于推送和IM | 物联网                   |
| 功耗     | 高               | 低                       |
| 反向控制 | 可以             | 非长链接，不适合反向控制 |

两者都适合物联网场景：
- MQTT：不需要考虑功耗的场景但需要被实时控制，比如：充电桩、快递柜；
- CoAP：只是上报数据，或者对功耗敏感，比如：水表、燃气表；

## LwM2M
Lightweight M2M，一个专门为资源受限的物联网设备设计的协议，OMA（一家国际组织）基于OMA-DM（远程管理移动设备）协议基础上设计，是一组轻量级协议，是一套C/S通信机制

### 逻辑实体

1. Server服务器
2. Client 客户端：负责执行服务器的命令和上报执行结果
3. BootStrap：引导服务器，负责配置客户端

### 逻辑接口

1. **Device Discovery and Registration**：让客户端注册到服务器，并通知服务器客户端所支持的能力；
2. **Bootstrap**：配置客户端，比如说Server的URL地址；
3. **Device Management and Service Enablement**：业务接口，Server发送指令给Client并收到回应
4. **Information Reporting**：Client上报数据，比如温度
	1. 事件出发
	2. 周期行

### 关系图

![](https://pic4.zhimg.com/80/v2-f0bfbefb099d118ec0c6de5835c5e73f_1440w.webp)

### 协议栈

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309022359146.png)

- **LwMM Objects**：每个对象对应客户端某个特定功能实体，LwM2M 规范定义了一下标准Objects，比如
urn:oma:lwm2m:oma:2; (LwM2M Server Object)
urn:oma:lwm2m:oma:3; (LwM2M Access Control Object)
每个object下可以有很多resource. 比如Firmware object可以有Firmware版本号，size等resource.
Vendor可以自己定义object

- **LwM2M Protocol**：定义一些逻辑操作，比如：Read、Write、Execute、Create or Delete.
- **CoAP**：作为传输层，底下可以是UDP（必须支持）或SMS（可选）
- **DTLS**：保证C/S间的安全性
#### 与CoAP关系
LwM2M的消息没有对称的反馈消息，由于LwM2M承载在CoAP协议上，使用CoAP的get、post、put、delete方式，对于相应消息成功或失败的反馈是通过CoAP协议本身的交互来实现的。LwM2M载荷支持四种格式 plain text、Opaque、TLV、JSON,这四种协议要求服务器端必须都要支持，而在客户端必须支持TLV格式。

## 参考链接
[[物联网IoT终端设备如何选择接入协议]]

# 近场通讯

选择因素：范围、数据要求、安全、电力需求、电池寿命
