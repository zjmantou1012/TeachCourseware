# https与http的区别

Https在Http下多了一层ssl/TSL安全层；

通过ca申请证书；

多了一层ssl握手；

端口号为443

# Http1.1和Http1.0及2.0的区别？

## HTTP1.0和HTTP1.1区别

1. 缓存处理，提供了更多可选择的缓存头来控制缓存策略，比如if-Match、if-Unmodified
2. 带宽优化
3. 错误通知管理，新增了更多的错误状态响应码；
4. 增加Host头处理；
5. 长连接；

## HTTP2.0和HTTP1.1区别

1. 新的二进制格式
2. 多路复用
3. header压缩
4. 服务端推送（server push）

# HTTP请求慢的解决办法

1. 不通过DNS解析，直接访问IP
2. 解决连接无法复用
	1. 基于TCP的socket长链接
	2. http long-polling
		1. 增加服务端压力
		2. 需要考虑如何重建健康的连接通道
		3. 稳定性不好
		4. response可能被中间代理cache住
	3. http streaming：通过再server response的头部增加“Transfer Encoding:chuncked”来告诉客户端后续还有新的数据到来；
		1. 不会结束response
		2. 业务数据无法按照请求分割
	4. Websokect
3. 解决head of line blocking
	1. 队列的第一个数据包（队头）受阻而导致整列数据包受阻，使用http pipelining，确保几乎在同一时间把request发向了服务器

# Request和Response的协议组成

## Request

请求行（request line）、请求头部（header）、空行和请求数据

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310311439107.png)

## Response

状态行、消息报头、空行和响应正文

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310311440640.png)

# 缓存机制

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310311441949.png)

## 强制缓存

第一次请求服务端返回了缓存的过期时间（Expires与Cache-Control），没有过期就用缓存。

## 对比缓存

需要服务端参与判断是否使用缓存

通过第一次请求返回的缓存标识（Last-Modified/If-Modified-Since与Etag/If-None-Match）来判断，如果返回304，则继续使用缓存。

## 缓存标识

- Expires: 服务端返回的到期时间，有个时间校验问题，http1.1采用Cache-Control替代；
- Cache-Control：
	- private：客户端可以缓存；
	- public：户端和代理服务器都可缓存
	- max-age=xxx: 缓存的内容将在 xxx 秒后失效 
	- no-cache: 需要使用对比缓存来验证缓存数据。 
	- no-store: 所有内容都不会缓存，强制缓存，对比缓存都不会触发
- Etag/If-None-Match：
	- Etag优先级高于Last-Modified，标记资源是否修改；
	- If-None-Match会把上一次服务端返回的Etag传回去服务端做对比，不同则代表有修改。
- Last-Modified/If-Modified-Since：Last-Modified服务端返回的上次修改时间，If-Modified-Since是客户端再次请求是携带的上次返回的修改的时间。

# Http长链接

Http1.1默认长链接（默认Connection的值就是keep-alive），只TCP长链接，可以多个HTTP请求复用同一个TCP连接。

请求后保持一段时间，如果期间没有其他请求则断开，值是通过header设置。

# Https加密原理

加密算法：
- 对称加密：加密用的密钥和解密用的密钥是同一个，AES加密算法；
- 非对称加密：加密用的密钥称为公钥，解密用的密钥称为私钥， RSA 加密算法；

Hash加密算法：MD5, SHA1, SHA256

HTTPS = HTTP + SSL，加密算法在SSL中完成。

## SSL握手

1. 客户端和服务端建立 SSL 握手，客户端通过 CA 证书来确认服务端的身份；
2. 互相传递三个随机数，之后通过这随机数来生成一个密钥；
3. 互相确认密钥，然后握手结束；
4. 数据通讯开始，都使用同一个对话密钥来加解密；

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310311503449.png)


[需要更深的理解请点击这里](https://juejin.im/entry/6844903569343119367#comment "https://juejin.im/entry/6844903569343119367#comment")

# 相应码

1** 信息，服务器收到请求，需要请求者继续执行操作

2** 成功，操作被成功接收并处理

3** 重定向，需要进一步的操作以完成请求

4** 客户端错误，请求包含语法错误或无法完成请求

5** 服务器错误，服务器在处理请求的过程中发生了错误

# TCP与UDP

[[TCP与UDP]]

## 为什么要三次握手

![[TCP与UDP#为什么要三次握手]]

![[TCP与UDP#为什么需要四次挥手]]]

# Socket
[[Socket]]

## Http与Sokect对比

![[Socket#与HTTP对比]]


# IP报文

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310311555887.png)


版本：IP协议的版本，目前的IP协议版本号为4，下一代IP协议版本号为6。

首部长度：IP报头的长度。固定部分的长度（20字节）和可变部分的长度之和。共占4位。最大为1111，即10进制的15，代表IP报头的最大长度可以为15个32bits（4字节），也就是最长可为15*4=60字节，除去固定部分的长度20字节，可变部分的长度最大为40字节。

服务类型：Type Of Service。

总长度：IP报文的总长度。报头的长度和数据部分的长度之和。

标识：唯一的标识主机发送的每一分数据报。通常每发送一个报文，它的值加一。当IP报文长度超过传输网络的MTU（最大传输单元）时必须分片，这个标识字段的值被复制到所有数据分片的标识字段中，使得这些分片在达到最终目的地时可以依照标识字段的内容重新组成原先的数据。

标志：共3位。R、DF、MF三位。目前只有后两位有效，DF位：为1表示不分片，为0表示分片。MF：为1表示“更多的片”，为0表示这是最后一片。

片位移：本分片在原先数据报文中相对首位的偏移位。（需要再乘以8）

生存时间：IP报文所允许通过的路由器的最大数量。每经过一个路由器，TTL减1，当为0时，路由器将该数据报丢弃。TTL 字段是由发送端初始设置一个 8 bit字段.推荐的初始值由分配数字 RFC 指定，当前值为 64。发送 ICMP 回显应答时经常把 TTL 设为最大值 255。

协议：指出IP报文携带的数据使用的是那种协议，以便目的主机的IP层能知道要将数据报上交到哪个进程（不同的协议有专门不同的进程处理）。和端口号类似，此处采用协议号，TCP的协议号为6，UDP的协议号为17。ICMP的协议号为1，IGMP的协议号为2.

首部校验和：计算IP头部的校验和，检查IP报头的完整性。

源IP地址：标识IP数据报的源端设备。

目的IP地址：标识IP数据报的目的地址。

最后就是可变部分和数据部分。


# 网络流程解析

1、浏览器输入地址到返回结果发生了什么？

总体来说分为以下几个过程:
1. DNS解析，此外还有DNSy优化（DNS缓存、DNS负载均衡）
2. TCP连接
3. 发送HTTP请求
4. 服务器处理请求并返回HTTP报文
5. 浏览器解析渲染页面
6. 连接结束