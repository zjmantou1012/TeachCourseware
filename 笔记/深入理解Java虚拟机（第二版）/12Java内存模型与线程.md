---
author: zjmantou
title: 12Java内存模型与线程
time: 2023-12-11 周一
tags:
  - Java
  - Java虚拟机
  - 笔记
---
# Java内存模型

- 主内存：所有的变量都存储在主内存。
- 工作内存：每条线程有一条自己的工作内存。 

工作内存保存该线程使用的变量的主内存副本。 

线程对变量的所有操作都在工作内存中。 

线程间变量传递需通过主内存。 

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202312111628504.png)

## 内存间交互操作

1. lock（锁定）：作用于主内存的变量，它把一个变量标识为一条线程独占的状态。
2. unlock（解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
3. read（读取）：作用于主内存的变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用。
4. load（载入）：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。
5. use（使用）：作用于工作内存的变量，它把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作。
6. assign（赋值）：作用于工作内存的变量，它把一个从执行引擎接收的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
7. store（存储）：作用于工作内存的变量，它把工作内存中一个变量的值传送到主内存中，以便随后的write操作使用。
8. write（写入）：作用于主内存的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中。

操作规则：
- 不允许read和load、store和write操作之一单独出现。
- 不允许一个线程丢弃它最近的assign操作。
- 不允许一个线程无原因地（没有发生过任何assign操作）把数据从线程的工作内存同步回主内存中。
- 一个新的变量只能在主内存中“诞生”，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量，换句话说就是对一个变量实施use、store操作之前，必须先执行assign和load操作。
- 一个变量在同一个时刻只允许一条线程对其进行lock操作，但lock操作可以被同一条线程重复执行多次，多次执行lock后，只有执行相同次数的unlock操作，变量才会被解锁。
- 如果对一个变量执行lock操作，那将会清空工作内存中此变量的值，在执行引擎使用这个变量前，需要重新执行load或assign操作以初始化变量的值。
- 如果一个变量事先没有被lock操作锁定，那就不允许对它执行unlock操作，也不允许去unlock一个被其他线程锁定的变量。
- 对一个变量执行unlock操作之前，必须先把此变量同步回主内存中（执行store、write操作）。

## volatile特殊规则

- 可见性
- 禁止重排序

在操作volatile变量时需要满足以下规则：
- 只有当线程T对变量V执行的前一个动作是load的时候，线程T才能对变量V执行use动作；并且，只有当线程T对变量V执行的后一个动作是use的时候，线程T才能对变量V执行load动作。线程T对变量V的use动作可以认为是和线程T对变量V的load、read动作相关联的，必须连续且一起出现。
- 只有当线程T对变量V执行的前一个动作是assign的时候，线程T才能对变量V执行store动作；并且，只有当线程T对变量V执行的后一个动作是store的时候，线程T才能对变量V执行assign动作。线程T对变量V的assign动作可以认为是和线程T对变量V的store、write动作相关联的，必须连续且一起出现。
- 假定动作A是线程T对变量V实施的use或assign动作，假定动作F是和动作A相关联的load或store动作，假定动作P是和动作F相应的对变量V的read或write动作；与此类似，假定动作B是线程T对变量W实施的use或assign动作，假定动作G是和动作B相关联的load或store动作，假定动作Q是和动作G相应的对变量W的read或write动作。如果A先于B，那么P先于Q。

## 原子性、可见性、有序性

- 原子性：由Java内存模型来直接保证的原子性变量操作包括read、load、assign、use、store和write这六个。
- 当一个线程修改了共享变量的值时，其他线程能够立即得知这个修改。
- 有序性：线程内有序、线程间无序。

# 线程状态

- New
- Runnable
- Waiting
	- Object::wait()
	- Thread::join()
	- LockSupport::park()
- Time Waiting
	- sleep()
	- 有时间参数的Object::wait()
	- 有时间参数的Thread::join()
	- LockSupport::parkNanos()
- Block
- Terminated
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202312111701494.png)


# 当下线程的问题

用户线程切换的开销，上下文数据的保存与恢复。

# 协程核心

轻量，状态机