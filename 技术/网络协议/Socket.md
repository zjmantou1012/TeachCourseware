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

# 建立连接

套接字之间的连接过程分为三个步骤：服务器监听，客户端请求，连接确认。

- 服务器监听：服务器端套接字并不定位具体的客户端套接字，而是处于等待连接的状态，实时监控网络状态，等待客户端的连接请求。
- 客户端请求：指客户端的套接字提出连接请求，要连接的目标是服务器端的套接字。为此，客户端的套接字必须首先描述它要连接的服务器的套接字，指出服务器端- - 套接字的地址和端口号，然后就向服务器端套接字提出连接请求。
- 连接确认：当服务器端套接字监听到或者说接收到客户端套接字的连接请求时，就响应客户端套接字的请求，建立一个新的线程，把服务器端套接字的描述发 给客户端，一旦客户端确认了此描述，双方就正式建立连接。而服务器端套接字继续处于监听状态，继续接收其他客户端套!

# 断线重连

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310311548133.png)


# Cookie与Session的作用

- Session是在服务端保存的一个数据结构，用来跟踪用户的状态，这个数据可以保存在集群、数据库、文件中。
- Cookie是客户端保存用户信息的一种机制，用来记录用户的一些信息，也是实现Session的一种方式。

