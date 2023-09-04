
蓝牙Mesh网络是一个完整的全栈解决方案，它定义了从物理无线电层到应用层的所有技术规范。除了有助于实现更简单的产品开发和更高级的产品互操作性之外，这种全栈解决方案还可以实现更快、更顺畅的技术迭代与演进。
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309042019940.png)


# SIG Mesh基础知识
蓝牙核心官方文档：
[[Core_v5.3.pdf]]

## [什么是Mesh网络](https://www.jianshu.com/p/ce56f75284b8)

无线Mesh技术是一种与传统无线网络完全不同的新型无线网络技术。在传统的WLAN中，每个客户端均通过一条与接入点（AP）相连的无线链路访问网络，用户若要进行互相通信，必须首先访问一个固定的AP，这种网络结构称为单跳网络，而无线Mesh网络中，任何无线设备节点都可同时作为路由器，网络中的每个节点都能发送和接收信号，每个节点都能与一个或多个对等节点进行直接通信。

## 什么是蓝牙Mesh网络

[蓝牙Mesh系列文章汇总](https://blog.csdn.net/wanguofeng8023/article/details/119522923)



蓝牙Mesh传播的方式：基于低功耗蓝牙广播报文（Advertising）实现，这是一种基于洪泛（Managed-flooding）的信息传递机制。
	**当蓝牙mesh网络中的一个节点需要向另一个节点发送消息时，它会广播一条消息，所有收到这个消息的结点都接受并且转发这条消息，这样就可以保证目标结点只要在整个网络的覆盖范围内，就能收到这条消息**
采用了信息缓存（Message Cache）队列和TTL（Time To Live，信息寿命）字段这两种方案来避免信息被无限制地转发下去。
- Message cache：设备都会缓存收到消息的关键信息，以确定是否已经转发此消息，如果是就忽略此消息。Message Cache需要至少能缓存两条消息。
- Time to Live：每个消息都会包含一个TTL的值，来限制中继的次数，最大可中继126次。消息每转发一次TTL的值就减1，TTL值为1就不再转发。

## 蓝牙Mesh网络架构

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309042033812.png)

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309042145284.png)


- 模型（models）：
- 基础模型（foundation models）：
- 接入层（access layer）：
- 上层传输层（upper transport layer）：
- 底层传输层（lower transport layer）：
- 网络层（network layer）
- 承载层（bearer layer）：承载层定义了如何使用底层低功耗堆栈传输PDU。目前定义了两个承载层：广播承载层（Advertising Bearer）和GATT承载层。
- 低功耗蓝牙核心规范（Bluetooth Low Energy Core Specification）:

## 通信机制

以消息为中心的网络，通过发布/订阅模式
	设备可以将消息发送至特定地址(单播)或地址群(群组)，这些地址的名称和含义与用户能够理解的高级概念相对应，如“厨房灯”(Kitchen Lights)。这被称为“发布”(Publish)。
	设备经配置后，可接收由其他设备发送到特定地址的消息。这被称为“订阅”(Subscribe)。

## 地址类型

- **单播（unicast）地址0xxxxxxxxxxxxxxx**：可能出现在消息的源地址字段或目的地地址字段中，发送到单播地址的消息只能由一个节点元素处理。
- **虚拟（virtual）地址10xxxxxxxxxxxxxx**：与特定的UUID标签相关联的一组节点元素。
- **群组（group）地址11xxxxxxxxxxxxxx**：通常代表一个或多个节点中的多个节点元素
	- 1111111111111100, 所有代理节点
	- 1111111111111101, 所有朋友节点
	- 1111111111111110, 所有中继节点
	- 1111111111111111, 所有节点
- **未分配（unassigned）地址0000000000000000**：不会用于传递消息

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309042107075.png)

## 与其他无线Mesh对比的优势
- 采用分布式控制架构，能够实现更大规模，更高的可靠性和更低的成本
- 除了支持单个寻址外还支持基于群组的多播寻址方式（发布/订阅机制）。可显著降低网络上的消息传递流量，从而实现更大网络规模和更佳通信性能



# 参考文档
[蓝牙Mesh网络及通信机制剖析](https://zhuanlan.zhihu.com/p/576932068)***

