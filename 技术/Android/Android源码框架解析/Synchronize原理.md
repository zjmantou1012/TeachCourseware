---
author: zjmantou
title: Synchronize原理
time: 2023-10-19 周四
tags:
  - Android
  - 技术
---
特征：

同一时刻只有一个线程能够获得对象monitor，确保当前线程能执行到相应的同步逻辑，线程之间表现为互斥性，其他的线程锁在EntryList队列中排队；

锁优化：**通过局部的优化来提升系统整体的并发同步的效率**。

# synchronized、volatile以及CAS的区别

[https://blog.csdn.net/erge353729094/article/details/107700900](https://blog.csdn.net/erge353729094/article/details/107700900)

## **CAS（Compare And Swap）**

### 什么是CAS

非阻塞性原子性操作

synchronize是悲观锁策略，CAS是乐观锁策略，假设所有线程访问共享资源时都不会冲突；它是比较交换来鉴别线程是否出现冲突，出现冲突就重试当前操作直到没有冲突为止；

### CAS操作过程

包含三个值分别为：**V 内存地址存放的实际值；O 预期的值（旧值）；N 更新的新值**。当V和O相同时，也就是说旧值和内存中实际的值相同表明该值没有被其他线程更改过，即该旧值O就是目前来说最新的值了，自然而然可以将新值N赋值给V。反之，V和O不相同，表明该值已经被其他线程改过了则该旧值O不是最新版本的值了，所以不能将新值N赋给V，返回V即可。当多个线程使用CAS操作一个变量时，只有一个线程会成功，并成功更新，其余会失败。失败的线程会重新尝试，当然也可以选择挂起线程

### Synchronized VS CAS

元老级的Synchronized(未优化前)最主要的问题是：在存在线程竞争的情况下会出现线程阻塞和唤醒锁带来的性能问题，因为这是一种互斥同步（**阻塞同步**）。而CAS并不是武断的间线程挂起，当CAS操作失败后会进行一定的尝试，而非进行耗时的挂起唤醒的操作，因此也叫做**非阻塞同步**。这是两者主要的区别

### CAS的应用场景

1. 在J.U.C包中利用CAS实现类很多，可以说支撑起整个concurrency包的实现；
2. 在Lock实现中会有CAS改变state的变量；
3. 在atomic中的实现类也几乎都是用CAS实现；
4. jdk1.8之后的ConcurrentHashmap。

**CAS的问题**

1. ABA问题：A->B->A；解决方法：添加一个版本号，A->B->A变成了1A->2B->3C；
2. 自旋时间过长：使用CAS时非阻塞同步，也就是说不会将线程挂起，会自旋（无非就是一个死循环）进行下一次尝试，如果这里自旋时间过长对性能是很大的消耗。如果JVM能支持处理器提供的pause指令，那么在效率上会有一定的提升；
3. 只能保证一个共享变量的原子操作。

当对一个共享变量执行操作时CAS能保证其原子性，如果对多个共享变量进行操作,CAS就不能保证其原子性。有一个解决方案是利用对象整合多个共享变量，即一个类中的成员变量就是这几个共享变量。然后将这个对象做CAS操作就可以保证其原子性。atomic中提供了AtomicReference来保证引用对象之间的原子性。

# 各种锁的比较

![](https://cdn.nlark.com/yuque/0/2022/jpeg/26044650/1649065725527-40b2da3b-d6e9-470f-a10f-f257b3c85d13.jpeg)

作者：你听___

链接：https://juejin.cn/post/6844903600334831629

来源：稀土掘金

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


# Synchronized实现原理

同步代码块

```Java
package com.paddx.test.concurrent;
public class SynchronizedDemo {
    public void method() {
        synchronized (this) {
            System.out.println("Method 1 start");
        }
    }
}
```
这段代码得到的反编译后的代码：

![sync.png](https://upload-images.jianshu.io/upload_images/2062729-b98084591219da8c.png)


monitorenter：每个对象都是一个监视器锁（monitor）。当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权：
1. 如果monitor的进入数为0，则该线程进入monitor，然后进入数为1，该线程即为monitor的所有者；
2. 如果线程已经占有该monitor，只是重新进入，则monitor的进入数+1；
3. 如果其他线程已经占用了monitor，则该线程进入阻塞状态，知道monitor的进入数为0，再重新尝试获取monitor的所有权。

monitorexit：执行这个的线程必须时objectref所对应的所有者，指令执行时，monitor的进入数减1，如果减1后进入数为0，那线程退出monitor，就不再是这个monitor的所有者。其他被这个monitor阻塞的线程可以尝试去获取这个monitor的所有权。
>monitorexit指令出现了两次，第一次为同步推出释放锁，第二次为发生异步退出释放锁。


同步方法：
```Java
package com.paddx.test.concurrent;

public class SynchronizedMethod {
    public synchronized void method() {
        System.out.println("Hello World!");
    }
}
```

![syncmethod.png](https://upload-images.jianshu.io/upload_images/2062729-8b7734120fae6645.png)

同步方法中是检查是否设置了ACC_SYNCHRONIZED标识符，如果设置了，执行的线程先获取monitor，执行完之后释放，在方法执行期间，其他任何线程都无法再获得同一个monitor对象。

## 实现原理

Synchronized底层是通过一个monitor的对象来完成，同一时刻只有一个线程能够获得对象monitor，确保当前线程能执行到相应的同步逻辑，线程之间表现为互斥性，其他的线程锁在EntryList队列中排队。

wait、notify等方法也是依赖于monitor对象，所以这两个方法也只能在同步代码块中调用。


# 参考链接

[深入分析Synchronized原理(阿里面试题) ](https://www.cnblogs.com/aspirant/p/11470858.html)
