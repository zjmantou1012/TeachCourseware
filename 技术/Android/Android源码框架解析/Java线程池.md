---
author: zjmantou
title: Java线程池
time: 2023-10-31 周二
tags:
  - Java
  - 技术
---
# 线程池的复用

addWorker方法，取任务的方法：
- firstTask，指定第一个工作任务
- getTask：死循环，直到能从队列中取出任务。

# 信号量

Semaphore

- 资源并发控制
- 控制线程并发数
- 实现互斥锁
- 控制任务流量


