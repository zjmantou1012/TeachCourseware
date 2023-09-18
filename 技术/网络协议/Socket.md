---
author: zjmantou
title: Socket
time: 2023-09-15 周五
tags:
  - 网络协议
  - socket
---
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309151658662.png)

[[TCP与UDP]]

- 即套接字，**是一个对 TCP / IP协议进行封装 的编程调用接口（API）**
	1. 通过Socket，我们才能在Android平台上通过TCP/IP协议进行开发；
	2. Socket不是一种协议，而是一个编程调用接口（API），属于传输层（主要解决数据如何在网络中传输）。
- 成对出现，
```apache
Socket ={(IP地址1:PORT端口号)，(IP地址2:PORT端口号)}

```
# 原理
使用类型：
- 流套接字（`streamsocket`） ：基于 `TCP`协议，采用 **流的方式** 提供可靠的字节流服务
- 数据报套接字(`datagramsocket`)：基于 `UDP`协议，采用 **数据报文** 提供数据打包发送的服务。
![socket原理](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309151704150.png)

# 与HTTP对比

- socket属于传输层：解决的是数据如何在网络中传输
- HTTP属于应用层：解决的是数据如何包装
- HTTP采用请求/响应的方式
- socket采用服务器主动发送数据方式

# 使用步骤
```Java

// 步骤1：创建客户端 & 服务器的连接

    // 创建Socket对象 & 指定服务端的IP及端口号 
    Socket socket = new Socket("192.168.1.32", 1989);  

    // 判断客户端和服务器是否连接成功  
    socket.isConnected());


// 步骤2：客户端 & 服务器 通信
// 通信包括：客户端 接收服务器的数据 & 发送数据 到 服务器

    <-- 操作1：接收服务器的数据 -->

            // 步骤1：创建输入流对象InputStream
            InputStream is = socket.getInputStream() 

            // 步骤2：创建输入流读取器对象 并传入输入流对象
            // 该对象作用：获取服务器返回的数据
            InputStreamReader isr = new InputStreamReader(is);
            BufferedReader br = new BufferedReader(isr);

            // 步骤3：通过输入流读取器对象 接收服务器发送过来的数据
            br.readLine()；


    <-- 操作2：发送数据 到 服务器 -->                  

            // 步骤1：从Socket 获得输出流对象OutputStream
            // 该对象作用：发送数据
            OutputStream outputStream = socket.getOutputStream(); 

            // 步骤2：写入需要发送的数据到输出流对象中
            outputStream.write（（"Carson_Ho"+"\n"）.getBytes("utf-8")）；
            // 特别注意：数据的结尾加上换行符才可让服务器端的readline()停止阻塞

            // 步骤3：发送数据到服务端 
            outputStream.flush();  


// 步骤3：断开客户端 & 服务器 连接

             os.close();
            // 断开 客户端发送到服务器 的连接，即关闭输出流对象OutputStream

            br.close();
            // 断开 服务器发送到客户端 的连接，即关闭输入流读取器对象BufferedReader

            socket.close();
            // 最终关闭整个Socket连接

```

