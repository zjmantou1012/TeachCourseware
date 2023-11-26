https://blog.csdn.net/skdin/article/details/126593812

[Android：IdleHandler的简单理解和使用](https://blog.csdn.net/JMW1407/article/details/129133416)



Looper：

创建好的looper会放在ThreadLocal中，以健值对的形式存储，key为当前线程


插入消息：

通过Messagequeue的enqueueMessage将Message加入到消息队列中，通过when比较，遍历找出插入位置；

消息发送：

调用sendMessageDelayed(),获取Messsage后把Runnable复制给callback属性

Looper.loop()

无限循环操作拿到头消息，调用message.next.dispatchMessage()发送

注：只有出发到了时间点才会发送

![](https://cdn.nlark.com/yuque/0/2022/jpeg/26044650/1649065768416-06c2d449-65d3-44b3-b077-102e88c21c18.jpeg)

问题：

1、为什么Activity中使用Handler不需要创建Looper

2、为什么Activity中使用Handler不需要使用looper.loop()

因为在ActivityThread中已经调用了Looper.prepareMainLooper()创建，且用了looper.loop()，中间创建了MainThreadHandler；

3、Looper.loop()是个无限循环，且会阻塞线程，那么主线程为什么看起来没有阻塞呢

补充

HandlerThread：

HandlerThread继承自Thread所以他拥有Thread能力，与之不同的是HandlerThread内部自建创建了一个Handler，提供了Looper

  
  

Message.obtain()

从消息池中返回一个消息，且不会创建对象

Message什么时候加到池中？当Mesage被looper分发完后，会调用recycleUnchecked（）回收没有在使用的Message对象

怎么维护消息池

消息池也是链表结构，在消息被消费后消息池会执行回收操作，将该消息内部数据清空然后添加消息链表最前面。

IdleHandler：

当MessageQueue中无可处理的Message时回调；

作用：UI线程处理完所有VIew事务后，回调一些额外的操，在next()方法中调用了这个

1、Handler问题三连：是什么？有什么用？为什么要用，不用行不行？

2、Android UI更新机制(GUI) 为何设计成了单线程的？

3、真的只能在主(UI)线程中更新UI吗？

不是，只有创建视图层次结构的原始线程才能触摸其视图，比如在子线程创建UI并操作更新可以，

另外在onCreate中创建线程立即执行更新UI也是可以的，因为ViewRootImpl在onCreate的时候还没有创建，reResume（）时才创建，调用View.requestLayout()最终调用的是ViewRootImpl.requestLayout()，才能走到checkThread()报错

4、真的不能在主(UI)线程中执行网络操作吗？

把StrictMode关闭可以，但不建议，容易引起ANR

5、Handler怎么用？

6、为什么建议使用Message.obtain()来创建Message实例？

7、为什么子线程中不可以直接new Handler()而主线程中可以？

8、主线程给子线程的Handler发送消息怎么写？

9、HandlerThread实现的核心原理？

10、当你用Handler发送一个Message，发生了什么？

11、Looper是怎么拣队列里的消息的？

12、分发给Handler的消息是怎么处理的？

13、IdleHandler是什么？

14、Looper在主线程中死循环，为啥不会ANR？

当系统受到因用户操作产生的通知时，会通过 **Binder** 方式跨进程通知 **ApplicationThread**;

它通过**Handler机制**，往 **ActivityThread** 的 **MessageQueue** 中插入消息，唤醒了主线程；

**queue.next**() 能拿到消息了,然后 **dispatchMessage** 完成事件分发；

15、Handler泄露的原因及正确写法

**非静态内部类会持有一个外部类的隐式引用**，可能会造成外部类无法被GC； 比如这里的Handler，就是非静态内部类，它会持有Activity的引用从而导致Activity无法正常释放

![](https://cdn.nlark.com/yuque/0/2022/png/26044650/1659582738691-7332f50b-61c0-408c-bd18-07d1f4f3f421.png)

解决方法：静态内部类，加上弱引用

```
private static class MyHandler extends Handler {
    //创建一个弱引用持有外部类的对象
    private final WeakReference<MainActivity> content;

    private MyHandler(MainActivity content) {
        this.content = new WeakReference<MainActivity>(content);
    }

    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
        MainActivity activity= content.get();
        if (activity != null) {
            switch (msg.what) {
                case 0: {
                    activity.notifyUI();
                }
            }
        }
    }
}
```

16、Handler中的同步屏障机制

17、Android 11 Handler相关变更

  
  

作者：coder_pig

链接：https://juejin.cn/post/6844904150140977165

来源：稀土掘金

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。