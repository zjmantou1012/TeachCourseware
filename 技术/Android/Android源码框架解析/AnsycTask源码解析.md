---
author: zjmantou
title: AnsycTask源码解析
time: 2023-10-24 周二
tags:
  - Android
  - 源码
---

# 3.0版本之前

使用ThreadPoolExecutor，核心线程数5个，最大线程数128个，非核心线程数等待时长1s，阻塞队列为LinkedBlockingQueue，容量为10.

缺点：  
最多能同时容纳138个任务，就会执行默认的饱和策略。  

# 7.0版本

两个线程池组成；
- SerialExecutor：主要用来处理排队，将任务串行之行，保证一个时间段内只有一个任务之行；
- 默认的线程池：核心线程和最大线程数根据cpu核数计算出来。阻塞队列用的是LinkedBlockingQueue，容量为128。

## WorkerRunnable

实现了Callable，封装了参数传递给FutureTask。

## FutureTask

可管理的异步任务；实现了Runnable和Callable，可以给Executor执行，也可以直接调用run执行。