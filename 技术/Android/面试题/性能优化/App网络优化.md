---
author: zjmantou
title: App网络优化
time: 2023-11-03 周五
tags:
  - Android
  - 性能优化
---
# 1、移动端获取网络数据优化的几个点

- 1、连接复用：节省连接建立时间，如开启 keep-alive。于Android来说默认情况下HttpURLConnection和HttpClient都开启了keep-alive。只是2.2之前HttpURLConnection存在影响连接池的Bug。
    
- 2、请求合并：即将多个请求合并为一个进行请求，比较常见的就是网页中的CSS Image Sprites。如果某个页面内请求过多，也可以考虑做一定的请求合并。
    
- 3、减少请求数据的大小：对于post请求，body可以做gzip压缩的，header也可以做数据压缩(不过只支持http 2.0)。 返回数据的body也可以做gzip压缩，body数据体积可以缩小到原来的30%左右（也可以考虑压缩返回的json数据的key数据的体积，尤其是针对返回数据格式变化不大的情况，支付宝聊天返回的数据用到了）。
    
- 4、根据用户的当前的网络质量来判断下载什么质量的图片（电商用的比较多）。
    
- 5、使用HttpDNS优化DNS：DNS存在解析慢和DNS劫持等问题，DNS 不仅支持 UDP，它还支持 TCP，但是大部分标准的 DNS 都是基于 UDP 与 DNS 服务器的 53 端口进行交互。HTTPDNS 则不同，顾名思义它是利用 HTTP 协议与 DNS 服务器的 80 端口进行交互。不走传统的 DNS 解析，从而绕过运营商的 LocalDNS 服务器，有效的防止了域名劫持，提高域名解析的效率。
    

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311031554339.png)


[参考文章](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F940be2e758ee "https://www.jianshu.com/p/940be2e758ee")

# 2、[客户端网络安全实现](https://link.juejin.cn/?target=http%3A%2F%2Fmrpeak.cn%2Fblog%2Fencrypt%2F "http://mrpeak.cn/blog/encrypt/")

# 3、设计一个网络优化方案，针对移动端弱网环境。