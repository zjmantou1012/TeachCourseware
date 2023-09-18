---
author: zjmantou
title: WebSockets
time: 2023-09-14 周四
tags:
  - 网络协议
  - 通信协议
---
`关键词`：应用层协议、基于 TCP、全双工通信、一次握手、持久连接、双向数据传输

# 特点
- 头部信息比较少，通常只有2bytes
- 服务器推送
- 兼容性好
- 基于TCP应用层协议
- 可以发送文本，也可以发送二进制数据
- 没有同源限制，客户端可以与任意服务器通信；
- 协议标识符是 ws（如果加密，则为 wss），服务器网址就是 URL；
- 支持扩展：ws 协议定义了扩展，用户可以扩展协议，或者实现自定义的子协议（比如支持自定义压缩算法等）；
## 优点
- **实时性：** Websocket支持服务器主动向客户端推送消息，使得客户端能够实时接收到服务器的事件和数据变化。
- **双向性：** Websocket支持全双工通信，即客户端和服务器可以同时发送和接收数据。
- **节约资源：** 相比于轮询机制，Websocket只需要建立一次连接即可实现实时通信，这样可以减少服务器的压力和网络流量。
- **兼容性：** Websocket 协议能够支持所有主流的浏览器和移动设备。

# 业务场景
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309141935651.png)


# 与MQTT区别

## MQTT
通常用于 IOT 设备上(作为MQTT Client)，基于 TCP 有一套自己的协议栈格式。MQTT Server[也称为 MQTT broker]通常在 PC 上。// blog.csdn.net/benhuo93111… MQTT Client 和 MQTT Server 通常扮演多对多的角色。 一个 Client 发布消息，多个 Client 将会收到该消息。

## WebSockets
通常用户 PC 上，Websocket也是基于 TCP 协议的，同时借用了HTTP的协议来完成一部分握手。 主要解决 HTTP 协议中一个 request 对应一个 response 的尴尬。(http server 不能主动发送消息给 http client) 通过 HTTP 完成 websocket 的握手过程，接着按照 websocket 协议进行通讯。


作者：machiming
链接：https://juejin.cn/post/6844903598204125192
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


# 与HTTP区别
- HTTP无状态，请求/响应的方式
- WebSocket：双向通信，只需建立连接，适合实时通信

## 区别总结

- **连接方式不同**： HTTP 是一种单向请求-响应协议，每次请求需要重新建立连接，而 WebSocket 是一种双向通信协议，使用长连接实现数据实时推送。
- **数据传输方式不同**： HTTP 协议中的数据传输是文本格式的，而 WebSocket 可以传输文本和二进制数据。
- **通信类型不同**： HTTP 主要用于客户端和服务器之间的请求和响应，如浏览器请求网页和服务器返回网页的 HTML 文件。WebSocket 可以实现双向通信，常常用于实时通信场景。
- **性能方面不同**： 由于 HTTP 的每次请求都需要建立连接和断开连接，而 WebSocket 可以在一次连接上进行多次通信，WebSocket 在性能上比 HTTP 有优势。

# 与TCP区别
WebSocket 和 HTTP 都是基于 TCP 协议的应用层协议。
- **层次结构：** WebSocket 是应用层协议，而 TCP 是传输层协议。
- **协议特点：** TCP 是一种面向连接的协议，使用三次握手建立连接，提供可靠的数据传输。而 WebSocket 是一种无状态的协议，使用 HTTP 协议建立连接，可以进行双向通信，WebSocket 的数据传输比 TCP 更加轻量级。
- **数据格式：** TCP 传输的数据需要自定义数据格式，而 WebSocket 可以支持多种数据格式，如 **[JSON](https://link.juejin.cn?target=https%3A%2F%2Fapifox.com%2Fapiskills%2Fwhat-is-json%2F "https://apifox.com/apiskills/what-is-json/")**、XML、二进制等。WebSocket 数据格式化可以更好的支持 Web 应用开发。

**连接方式：** TCP 连接的是物理地址和端口号，而 WebSocket 连接的是 URL 地址和端口号。


# 与Socket区别
- 协议不同：Socket是传输层TCP协议；WebSocket是基于HTTP协议，通过握手来实现；
- 持久化连接：Socket是短连接，WebSocket是长连接
- 效率：Socket没有HTTP头信息，效率更高
- 应用场景：
	- Socket：实时传输数据，例如在线游戏、聊天室等需要快速交换数据的场景
	-  Websocket 适用于需要长时间保持连接的场景，例如在线音视频、远程控制等。


作者：Apifox
链接：https://juejin.cn/post/7231430726430949437
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

# 参考链接
[WebSocket｜概念、原理、用法及实践](https://juejin.cn/post/7086021621542027271?searchId=20230914193328CA839A1ADDB5344302F6)

