---
author: zjmantou
title: Okhttp源码框架解析
time: 2023-11-07 周二
tags:
  - Android
  - 技术
  - 源码
---

# 源码解析

网络请求是从RealCall类的enqueue方法开始的，而实际是调用到了Dispatcher类的enqueue方法；

## Dispatcher任务调度

```Java

//最大并发数
private int maxRequests = 64;  
//每个主机的最大请求数
private int maxRequestsPerHost = 5;

/** 消费者线程池. 默认的线程池类似CachedThreadPool，适合执行大量的耗时比较少的任务 */  
private ExecutorService executorService;  
  
/** 将要运行的异步请求队列. */  
private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();  
  
/** 正在运行的异步请求队列 */  
private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();  
  
/** 正在运行的同步请求队列 */  
private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();

```

### enqueue方法

```Java
synchronized void enqueue(AsyncCall call) {  
  if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost) {  
    runningAsyncCalls.add(call);  
    executorService().execute(call);  
  } else {  
    readyAsyncCalls.add(call);  
  }
}
```

当正在运行的异步请求数量小于64，且正在运行的请求主机数小于5时，把请求加载到runningAsyncCalls中并子啊线程池中执行，否则就加入到readyAsyncCalls中进行缓存等待；

线程池执行的是一个AsyncCall，它是RealCall的内部类，实现了execute方法：

```Java
 @Override protected void execute() {  
    boolean signalledCallback = false;  
    try {  
      Response response = getResponseWithInterceptorChain(forWebSocket);  
      if (canceled) {  
        signalledCallback = true;  
        responseCallback.onFailure(RealCall.this, new IOException("Canceled"));  
      } else {  
        signalledCallback = true;  
        responseCallback.onResponse(RealCall.this, response);  
      }    } catch (IOException e) {  
      if (signalledCallback) {  
        // Do not signal the callback twice!  
        logger.log(Level.INFO, "Callback failure for " + toLoggableString(), e);  
      } else {  
        responseCallback.onFailure(RealCall.this, e);  
      }    } finally {  
      client.dispatcher().finished(this);  
    }  }}
```

Dispatcher中的finish方法：

```Java

synchronized void finished(AsyncCall call) {  
  if (!runningAsyncCalls.remove(call)) throw new AssertionError("AsyncCall wasn't running!");  
  promoteCalls();  
}

private void promoteCalls() {  
  if (runningAsyncCalls.size() >= maxRequests) return; // Already running max capacity.  
  if (readyAsyncCalls.isEmpty()) return; // No ready calls to promote.  
  
  for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {  
    AsyncCall call = i.next();  
  
    if (runningCallsForHost(call) < maxRequestsPerHost) {  
      i.remove();  
      runningAsyncCalls.add(call);  
      executorService().execute(call);  
    }  
    if (runningAsyncCalls.size() >= maxRequests) return; // Reached max capacity.  
  }  
}

```

将此次方法从队列中移除，然后从readyAsyncCalls中取出下一个请求加入到runningAsyncCalls中，并交给线程池处理；

## 拦截器

getResponseWithInterceptorChain方法

高版本拦截器中默认做了五个拦截器的处理：
- retryAndFollowUpInterceptor：失败重试和重定向
- BridgeInterceptor：必要的请求头添加和移除必要的响应头
- CacheInterceptor：负责读取缓存直接返回、更新缓存
- ConnectInterceptor：负责和服务器建立连接
- CallServerInterceptor：负责向服务器发送请求数据、从服务器读取响应数据

### 缓存处理CacheInterceptor

缓存拦截器会根据请求的信息和缓存的响应的信息来判断是否存在缓存可用，如果有可以使用的缓存，那么就返回该缓存给用户，否则就继续使用责任链模式来从服务器中获取响应。当获取到响应的时候，又会把响应缓存到磁盘上面。
[[Http#缓存机制]]

### ConnectInterceptor连接池

- 判断当前的连接是否可以使用：流是否已经被关闭，并且已经被限制创建新的流；
- 如果当前的连接无法使用，就从连接池中获取一个连接；
- 连接池中也没有发现可用的连接，创建一个新的连接，并进行握手，然后将其放到连接池中。

#### ConnectionPool连接管理

遍历寻找一个空闲时间最长的连接，根据该连接的闲置时长和最大连接数等参数来决定是否清理掉该连接，如果还是需要等待一段时间才清理，则返回这个时间差，在这个时间段之后清理。

# 整体流程图

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311071532775.png)


# 参考链接

[Android主流三方库源码分析（一、深入理解OKHttp源码）](https://juejin.cn/post/6844903631909552135)



