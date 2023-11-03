---
author: zjmantou
title: ReenTrantLock原理
time: 2023-11-01 周三
tags:
  - Java
  - 技术
  - 源码
---
基于AQS实现的独占式并发[[Java并发包基石——AQS]]，

# 公平锁与非公平锁

公平锁,顾名思义，意指锁的获取策略相对公平，当多个线程在获取同一个锁时，必须按照锁的申请时间来依次获得锁，排排队，不能插队；非公平锁则不同，当锁被释放时，等待中的线程均有机会获得锁。synchronized是非公平锁，ReentrantLock默认也是非公平的，但是可以通过带boolean参数的构造方法指定使用公平锁，但非公平锁的性能一般要优于公平锁。

# 源码分析

默认非公平锁

内部定义了三个重要的静态内部类，Sync，NonFairSync，FairSync。Sync作为ReentrantLock中公用的同步组件，继承了AQS（要利用AQS复杂的顶层逻辑嘛，线程排队，阻塞，唤醒等等）；NonFairSync和FairSync则都继承Sync，调用Sync的公用逻辑，然后再在各自内部完成自己特定的逻辑（公平或非公平）。

## 重入性


#### 非公平锁
##### 具体源码

```Java
static final class NonfairSync extends Sync {//继承Sync
        private static final long serialVersionUID = 7316153563782823691L;
        /** 获取锁 */
        final void lock() {
            if (compareAndSetState(0, 1))//CAS设置state状态，若原值是0，将其置为1
                setExclusiveOwnerThread(Thread.currentThread());//将当前线程标记为已持有锁
            else
                acquire(1);//若设置失败，调用AQS的acquire方法，acquire又会调用我们下面重写的tryAcquire方法。这里说的调用失败有两种情况：1当前没有线程获取到资源，state为0，但是将state由0设置为1的时候，其他线程抢占资源，将state修改了，导致了CAS失败；2 state原本就不为0，也就是已经有线程获取到资源了，有可能是别的线程获取到资源，也有可能是当前线程获取的，这时线程又重复去获取，所以去tryAcquire中的nonfairTryAcquire我们应该就能看到可重入的实现逻辑了。
        }
        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);//调用Sync中的方法
        }
    }

final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();//获取当前线程
            int c = getState();//获取当前state值
            if (c == 0) {//若state为0，意味着没有线程获取到资源，CAS将state设置为1，并将当前线程标记我获取到排他锁的线程，返回true
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {//若state不为0，但是持有锁的线程是当前线程
                int nextc = c + acquires;//state累加1
                if (nextc < 0) // int类型溢出了
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);//设置state，此时state大于1，代表着一个线程多次获锁，state的值即是线程重入的次数
                return true;//返回true，获取锁成功
            }
            return false;//获取锁失败了
        }
```

##### 具体流程

1. 先获取state值，若为0，意味着此时没有线程获取到资源，CAS将其设置为1，设置成功则代表获取到排他锁了；
2. 若state大于0，肯定有线程已经抢占到资源了，此时再去判断是否就是自己抢占的，是的话，state累加，返回true，重入成功，state的值即是线程重入的次数；
3. 其他情况，则获取锁失败。

#### 公平锁

##### 具体源码

```Java

static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);//直接调用AQS的模板方法acquire，acquire会调用下面我们重写的这个tryAcquire
        }

        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();//获取当前线程
            int c = getState();//获取state值
            if (c == 0) {//若state为0，意味着当前没有线程获取到资源，那就可以直接获取资源了吗？NO!这不就跟之前的非公平锁的逻辑一样了嘛。看下面的逻辑
                if (!hasQueuedPredecessors() &&//判断在时间顺序上，是否有申请锁排在自己之前的线程，若没有，才能去获取，CAS设置state，并标记当前线程为持有排他锁的线程；反之，不能获取！这即是公平的处理方式。
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {//重入的处理逻辑，与上文一致，不再赘述
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }

public final boolean hasQueuedPredecessors() {
        Node t = tail; // 尾结点
        Node h = head;//头结点
        Node s;
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());//判断是否有排在自己之前的线程
    }
```

##### 具体流程

可以看到，公平锁的大致逻辑与非公平锁是一致的，不同的地方在于有了!hasQueuedPredecessors()这个判断逻辑，即便state为0，也不能贸然直接去获取，要先去看有没有还在排队的线程，若没有，才能尝试去获取，做后面的处理。反之，返回false，获取失败。


### 释放锁

```Java

protected final boolean tryRelease(int releases) {
            int c = getState() - releases;//减去1个资源
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            //若state值为0，表示当前线程已完全释放干净，返回true，上层的AQS会意识到资源已空出。若不为0，则表示线程还占有资源，只不过将此次重入的资源的释放了而已，返回false。
            if (c == 0) {
                free = true;//
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
```

# 总结

ReentrantLock是一种可重入的，可实现公平性的互斥锁，它的设计基于AQS框架，可重入和公平性的实现逻辑都不难理解，每重入一次，state就加1，当然在释放的时候，也得一层一层释放。至于公平性，在尝试获取锁的时候多了一个判断：是否有比自己申请早的线程在同步队列中等待，若有，去等待；若没有，才允许去抢占。　

# 参考链接

[ReentrantLock实现原理及源码分析](https://www.cnblogs.com/chengxiao/p/7255941.html)
