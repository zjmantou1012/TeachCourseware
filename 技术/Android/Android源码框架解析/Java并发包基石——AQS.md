---
author: zjmantou
title: Java并发包基石——AQS
time: 2023-11-01 周三
tags:
  - Java
  - 技术
  - 源码
---
AQS：**AbstractQueuedSynchronizer**，Java并发包中很多并发工具的基石，比如ReentrantLock、SemaPhore、ReentrantReadWriteLock，SynchronousQueue、FutureTask，是一个构建锁和同步器的框架。 

# 基本实现原理

# 基本定义

AQS使用一个int成员变量state同步状态，通过内置的FIFO队列来完成获取资源线程的排队工作。
```Java
    private volatile int state;//共享变量，使用volatile修饰保证线程可见性
```

## 同步方式

支持两种同步方式：
- 独占式；
- 共享式；
独占式如：ReentrantLock、共享式如Semaphore，CountDownLatch，组合式如ReentrantReadWriteLock。

## 实现方式

使用模板方法模式设计：
1. 继承**AbstractQueuedSynchronizer**并重写指定的方法（对于共享资源state的获取与释放）；
2. 将AQS组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法，可重写的方法：
	1. **protected boolean tryAcquire(int arg)** : 独占式获取同步状态，试着获取，成功返回true，反之为false；
	2. **protected boolean tryRelease(int arg)** ：独占式释放同步状态，等待中的其他线程此时将有机会获取到同步状态；
	3. **protected int tryAcquireShared(int arg)** ：共享式获取同步状态，返回值大于等于0，代表获取成功；反之获取失败；
	4. **protected boolean tryReleaseShared(int arg)**：**共享式释放同步状态，成功为true，失败为false**
	5. **protected boolean isHeldExclusively()** ： 是否在独占模式下被线程占用。

# 同步器代码实现

直接采用JDK官方文档中的小例子来说明问题

```Java
package juc;

import java.util.concurrent.locks.AbstractQueuedSynchronizer;

/**
 * Created by chengxiao on 2017/3/28.
 */
public class Mutex implements java.io.Serializable {
    //静态内部类，继承AQS
    private static class Sync extends AbstractQueuedSynchronizer {
        //是否处于占用状态
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }
        //当状态为0的时候获取锁，CAS操作成功，则state状态为1，
        public boolean tryAcquire(int acquires) {
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }
        //释放锁，将同步状态置为0
        protected boolean tryRelease(int releases) {
            if (getState() == 0) throw new IllegalMonitorStateException();
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }
    }
        //同步对象完成一系列复杂的操作，我们仅需指向它即可
        private final Sync sync = new Sync();
        //加锁操作，代理到acquire（模板方法）上就行，acquire会调用我们重写的tryAcquire方法
        public void lock() {
            sync.acquire(1);
        }
        public boolean tryLock() {
            return sync.tryAcquire(1);
        }
        //释放锁，代理到release（模板方法）上就行，release会调用我们重写的tryRelease方法。
        public void unlock() {
            sync.release(1);
        }
        public boolean isLocked() {
            return sync.isHeldExclusively();
        }
}
```

# 源码分析

AQS中的FIFO队列叫“CLH”队列，由双向列表的Node组成，AQS维护head和tail的两个指针

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311011425807.png)

当线程获取资源失败（比如tryAcquire时试图设置state状态失败），会被构造成一个结点加入CLH队列中，同时当前线程会被阻塞在队列中（通过LockSupport.park实现，其实是等待态）。当持有同步状态的线程释放同步状态时，会唤醒后继结点，然后此结点线程继续加入到对同步状态的争夺中。

## Node节点

AQS的静态内部类

```Java
static final class Node {
        /** waitStatus值，表示线程已被取消（等待超时或者被中断）*/
        static final int CANCELLED =  1;
        /** waitStatus值，表示后继线程需要被唤醒（unpaking）*/
        static final int SIGNAL    = -1;
        /**waitStatus值，表示结点线程等待在condition上，当被signal后，会从等待队列转移到同步到队列中 */
        /** waitStatus value to indicate thread is waiting on condition */
        static final int CONDITION = -2;
       /** waitStatus值，表示下一次共享式同步状态会被无条件地传播下去
        static final int PROPAGATE = -3;
        /** 等待状态，初始为0 */
        volatile int waitStatus;
        /**当前结点的前驱结点 */
        volatile Node prev;
        /** 当前结点的后继结点 */
        volatile Node next;
        /** 与当前结点关联的排队中的线程 */
        volatile Thread thread;
        /** ...... */
    }
```

## 独占式

### 获取同步状态--acquire()

#### 具体源码

```Java
 public final void acquire(int arg) {
         if (!tryAcquire(arg) &&
 acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
 selfInterrupt();
 }

private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);//构造结点
        //指向尾结点tail
        Node pred = tail;
        //如果尾结点不为空，CAS快速尝试在尾部添加，若CAS设置成功，返回；否则，eng。
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        enq(node);
        return node;
    }

private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { //如果队列为空，创建结点，同时被head和tail引用
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {//cas设置尾结点，不成功就一直重试
                    t.next = node;
                    return t;
                }
            }
        }
    }

final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {//死循环
                final Node p = node.predecessor();//找到当前结点的前驱结点
                if (p == head && tryAcquire(arg)) {//如果前驱结点是头结点，才tryAcquire，其他结点是没有机会tryAcquire的。
                    setHead(node);//获取同步状态成功，将当前结点设置为头结点。
                    p.next = null; // 方便GC
                    failed = false;
                    return interrupted;
                }
                // 如果没有获取到同步状态，通过shouldParkAfterFailedAcquire判断是否应该阻塞，parkAndCheckInterrupt用来阻塞线程
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        //获取前驱结点的wait值 
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)//若前驱结点的状态是SIGNAL，意味着当前结点可以被安全地park
            return true;
        if (ws > 0) {
        // ws>0，只有CANCEL状态ws才大于0。若前驱结点处于CANCEL状态，也就是此结点线程已经无效，从后往前遍历，找到一个非CANCEL状态的结点，将自己设置为它的后继结点
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {  
            // 若前驱结点为其他状态，将其设置为SIGNAL状态
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }

private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);//使用LockSupport使线程进入阻塞状态
        return Thread.interrupted();// 线程是否被中断过
}
    
```




#### 具体流程
1. tryAcquire获取同步状态，成功则直接返回；否则，进入下一环节；
2. 线程获取同步状态失败，就构造一个结点，加入同步队列中，这个过程要保证线程安全；
3. 加入队列中的结点线程进入自旋状态，若是老二结点（即前驱结点为头结点），才有机会尝试去获取同步状态；否则，当其前驱结点的状态为SIGNAL，线程便可安心休息，进入阻塞状态，直到被中断或者被前驱结点唤醒。
### 同步状态释放--release()

#### 具体源码
```Java
public final boolean release(int arg) {
        if (tryRelease(arg)) {//调用使用者重写的tryRelease方法，若成功，唤醒其后继结点，失败则返回false
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);//唤醒后继结点
            return true;
        }
        return false;
    }
    
private void unparkSuccessor(Node node) {
        //获取wait状态
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);// 将等待状态waitStatus设置为初始值0
        Node s = node.next;//后继结点
        if (s == null || s.waitStatus > 0) {//若后继结点为空，或状态为CANCEL（已失效），则从后尾部往前遍历找到一个处于正常阻塞状态的结点　　　　　进行唤醒
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);//使用LockSupprot唤醒结点对应的线程
    }
```


#### 具体流程

需要找到头结点的后继结点进行唤醒，若后继结点为空或处于CANCEL状态，从后向前遍历找寻一个正常的结点，唤醒其对应线程。

## 共享式

```Java
	protected int tryAcquireShared(int arg) {
        throw new UnsupportedOperationException();
    }
```

1. 当返回值大于0时，表示获取同步状态成功，同时还有剩余同步状态可供其他线程获取；
2. 当返回值等于0时，表示获取同步状态成功，但没有可用同步状态了；
3. 当返回值小于0时，表示获取同步状态失败。

### 获取同步状态--acquireShared　

#### 具体源码
```Java
	public final void acquireShared(int arg) {
	//返回值小于0，获取同步状态失败，排队去；获取同步状态成功，直接返回去干自己的事儿。
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
    }

private void doAcquireShared(int arg) {
        final Node node = addWaiter(Node.SHARED);//构造一个共享结点，添加到同步队列尾部。若队列初始为空，先添加一个无意义的傀儡结点，再将新节点添加到队列尾部。
        boolean failed = true;//是否获取成功
        try {
            boolean interrupted = false;//线程parking过程中是否被中断过
            for (;;) {//死循环
                final Node p = node.predecessor();//找到前驱结点
                if (p == head) {//头结点持有同步状态，只有前驱是头结点，才有机会尝试获取同步状态
                    int r = tryAcquireShared(arg);//尝试获取同步装填
                    if (r >= 0) {//r>=0,获取成功
                        setHeadAndPropagate(node, r);//获取成功就将当前结点设置为头结点，若还有可用资源，传播下去，也就是继续唤醒后继结点
                        p.next = null; // 方便GC
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&//是否能安心进入parking状态
                    parkAndCheckInterrupt())//阻塞线程
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

private void setHeadAndPropagate(Node node, int propagate) {
        Node h = head; // Record old head for check below
        setHead(node);
        if (propagate > 0 || h == null || h.waitStatus < 0) {
            Node s = node.next;
            if (s == null || s.isShared())
                doReleaseShared();
        }
    }

```

#### 具体流程

大体逻辑与独占式的acquireQueued差距不大，只不过由于是共享式，会有多个线程同时获取到线程，也可能同时释放线程，空出很多同步状态，所以当排队中的老二获取到同步状态，如果还有可用资源，会继续传播下去。

### 释放同步状态--releaseShared
#### 具体源码

```Java
public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            doReleaseShared();//释放同步状态
            return true;
        }
        return false;
    }

private void doReleaseShared() {
        for (;;) {//死循环，共享模式，持有同步状态的线程可能有多个，采用循环CAS保证线程安全
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;          
                    unparkSuccessor(h);//唤醒后继结点
                }
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                
            }
            if (h == head)              
                break;
        }
    }


```

#### 具体流程

共享模式，释放同步状态也是多线程的，此处采用了CAS自旋来保证。

# 参考链接

[Java并发包基石-AQS详解](https://www.cnblogs.com/chengxiao/p/7141160.html)
