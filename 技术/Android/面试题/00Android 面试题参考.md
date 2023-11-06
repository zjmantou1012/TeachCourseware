---
author: zjmantou
title: 00Android 面试题参考
time: 2023-10-31 周二
tags:
  - Android
  - 面试
---

# Java基础

## 图片压缩方案

![[图片压缩方案#基础知识#小结]]

- 质量压缩：在不改变图片尺寸的情况下，改变图片的存储体积。
	- compress
- 采样压缩：是降低图像尺寸，达到相同目的。
	- 邻近采样：inSampleSize
	- 双线性采样：createBitmap，可以使用小数，处理文字上效果更好
	- 双三次采样
	- Lanczos：计算量最大。


![[图片压缩方案#Android中图片压缩的方法介绍#小结]]

## 集合
[[Java集合]]

### Map
![[Map#ConcurrentHashMap#面试题]]

![[Map#**HashMap 1.7与1.8的区别**]]


如果key是int类型，可以使用SparseArray避免自动装箱的过程提高效率，不是int类型的也可以使用ArrayMap，但是查找和插入的时间复杂度比hashmap低。

![[Map#ArrayMap]]



![[Map#ArrayMap]]


# 序列化

![[序列化-Serializable和Parcelable]]

## 操作系统
[[操作系统面试题]]

## 数据库
[[数据库]]

## 设计模式

Java 中一般认为有23种设计模式，我们不需要所有的都会，但是其中常用的种设计模式应该去掌握。下面列出了所有的设计模式。要掌握的设计模式我单独列出来了，当然能掌握的越多越好。
总体来说设计模式分为三大类：

创建型模式，共五种：

工厂方法模式、抽象工厂模式、单例模式、建造者模式、原型模式。

结构型模式，共七种：

适配器模式、装饰器模式、代理模式、外观模式、桥接模式、组合模式、享元模式。

行为型模式，共十一种：

策略模式、模板方法模式、观者模式、迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、中介者模式、解释器模式。

[[Android开发者必须掌握的设计模式]]

## 代理
### 静态代理和动态代理的区别

在于代理类生成的时间不同

### 使用场景
静态代理：四大组件同AIDL与AMS进行跨进程通信
动态代理：retrofit、Hook


## HandlerThread
![[HandlerThread]]

## 幂等性

对同一个系统，使用同样的条件，一次请求和重复的多次请求对系统资源的影响是一致的。

比如客户付款，多次付款，只有一次相同结果

实现方式：使用token机制，每一次操作生成一个唯一性的凭证，也就是token。一个token在操作的每一个阶段只有一次执行权，一旦执行成功则保存执行结果。对重复的请求，返回同一个结果。例如平台上的订单ID就可以作为token。

## 内部类

### 什么是内部类？内部类的作用。

- 内部类可以有多个实例，每个实例都有自己的状态信息，并且与其他外围对象的信息相互独立。
- 在单个外围类中，可以让多个内部类以不同的方式实现同一个接口，或者继承同一个类。
- 创建内部类对象并不依赖于外围类对象的创建。
- 内部类并没有令人迷惑的“is-a”关系，他就是一个独立的实体。
- 内部类提供了更好的封装，除了该外围类，其他类都不能访问。。

### 匿名内部类

- Java的只能继承一个父类或实现一个接口，kotlin可以多个；
- 编译后名称：包名.OutClass1
- 匿名内部类默认持有外部的引用，可能会导致内存泄漏
- 由编译器构造而成

### 匿名内部类为什么只能访问外部的final变量

为了保证变量的一致性

匿名内部类编译后是生成单独的一个类，该类使用的变量是以构造函数参数的形式传入，如果不定义成final就可以在匿名内部类中随意修改。

匿名内部类的用法：
```Java
public class TryUsingAnonymousClass {
    public void useMyInterface() {
        final Integer number = 123;
        System.out.println(number);

        MyInterface myInterface = new MyInterface() {
            @Override
            public void doSomething() {
                System.out.println(number);
            }
        };
        myInterface.doSomething();

        System.out.println(number);
    }
}

```
编译后的结果：
```Java
class TryUsingAnonymousClass$1
        implements MyInterface {
    private final TryUsingAnonymousClass this$0;
    private final Integer paramInteger;

    TryUsingAnonymousClass$1(TryUsingAnonymousClass this$0, Integer paramInteger) {
        this.this$0 = this$0;
        this.paramInteger = paramInteger;
    }

    public void doSomething() {
        System.out.println(this.paramInteger);
    }
}

```

### 静态内部类、非静态内部类的理解？

- 静态内部类：只是为了降低包的深度，方便类的使用，静态内部类适用于包含在类当中，但又不依赖与外在的类，不用使用外在类的非静态属性和方法，只是为了方便管理类结构而定义。在创建静态内部类的时候，不需要外部类对象的引用。
- 非静态内部类：持有外部类的引用，可以自由使用外部类的所有变量和方法。



## String，StringBuffer，StringBuilder有哪些不同？

三者在执行速度方面的比较：StringBuilder >  StringBuffer  >  String

String每次变化一个值就会开辟一个新的内存空间

StringBuilder：线程非安全的

StringBuffer：线程安全的

对于三者使用的总结： 

1.如果要操作少量的数据用 String。

2.单线程操作字符串缓冲区下操作大量数据用 StringBuilder。

3.多线程操作字符串缓冲区下操作大量数据用 StringBuffer。

### String为什么不可变？

本质上是一个final的char[]数组，不可变性保证线程安全以及字符串常量池的实现。

## 抽象类和接口区别？

### 共同点

- 是上层的抽象层。
- 都不能被实例化。
- 都能包含抽象的方法，这些抽象的方法用于描述类具备的功能，但是不提供具体的实现。
### 区别

1. 在抽象类中可以写非抽象的方法，从而避免在子类中重复书写他们，这样可以提高代码的复用性，这是抽象类的优势，接口中只能有抽象的方法。
2. 多继承：一个类只能继承一个直接父类，这个父类可以是具体的类也可是抽象类，但是一个类可以实现多个接口。
3. 抽象类可以有默认的方法实现，接口根本不存在方法的实现。
4. -子类使用extends关键字来继承抽象类。如果子类不是抽象类的话，它需要提供抽象类中所有声明方法的实现。子类使用关键字implements来实现接口。它需要提供接口中所有声明方法的实现。
5. 构造器：抽象类可以有构造器，接口不能有构造器。
6. 和普通Java类的区别：除了你不能实例化抽象类之外，抽象类和普通Java类没有任何区别，接口是完全不同的类型。
7. 访问修饰符:抽象方法可以有public、protected和default修饰符，接口方法默认修饰符是public。你不可以使用其它修饰符。
8. main方法:抽象方法可以有main方法并且我们可以运行它接口没有main方法，因此我们不能运行它。
9. 速度:抽象类比接口速度要快，接口是稍微有点慢的，因为它需要时间去寻找在类中实现的方法。
10. 添加新方法:如果你往抽象类中添加新的方法，你可以给它提供默认的实现。因此你不需要改变你现在的代码。如果你往接口中添加方法，那么你必须改变实现该接口的类。

### 接口的意义？

规范、扩展、回调。

### 抽象类的意义?

为其子类提供一个公共的类型，封装子类中的重复内容，定义抽象方法，子类虽然有不同的实现 但是定义是一致的。

# 类加载机制

## 对象生命周期
### 1.创建阶段(Created)
JVM 加载类的class文件 此时所有的static变量和static代码块将被执行
加载完成后，对局部变量进行赋值（先父后子的顺序）
再执行new方法 调用构造函数
一旦对象被创建，并被分派给某些变量赋值，这个对象的状态就切换到了应用阶段。
### 2.应用阶段(In Use)
对象至少被一个强引用持有着。
### 3.不可见阶段(Invisible)
当一个对象处于不可见阶段时，说明程序本身不再持有该对象的任何强引用，虽然该这些引用仍然是存在着的。
简单说就是程序的执行已经超出了该对象的作用域了。
### 4.不可达阶段(Unreachable)
对象处于不可达阶段是指该对象不再被任何强引用所持有。
与“不可见阶段”相比，“不可见阶段”是指程序不再持有该对象的任何强引用，这种情况下，该对象仍可能被JVM等系统下的某些已装载的静态变量或线程或JNI等强引用持有着，这些特殊的强引用被称为”GC root”。存在着这些GC root会导致对象的内存泄露情况，无法被回收。
### 5.收集阶段(Collected)
当垃圾回收器发现该对象已经处于“不可达阶段”并且垃圾回收器已经对该对象的内存空间重新分配做好准备时，则对象进入了“收集阶段”。如果该对象已经重写了finalize()方法，则会去执行该方法的终端操作。
### 6.终结阶段(Finalized)
当对象执行完finalize()方法后仍然处于不可达状态时，则该对象进入终结阶段。在该阶段是等待垃圾回收器对该对象空间进行回收。
### 7.对象空间重分配阶段(De-allocated)
垃圾回收器对该对象的所占用的内存空间进行回收或者再分配了，则该对象彻底消失了，称之为“对象空间重新分配阶段。

[[技术/Android/面试题/Java类加载机制]]]

## 线程池
[[Java线程池]]

## Synchronized、volatile、ReenTrantLock
[[Synchronize原理]]

### 自旋锁

当获取的锁被占用时一直循环检测等待获取锁的机会

比较临界值很小的情况，不然长时间会影响整体性能

### 偏向锁

偏向锁就是一旦线程第一次获得了监视对象，之后让监视对象“偏向”这个线程，之后的多次调用则可以避免CAS操作，说白了就是置个变量，如果发现为true则无需再走各种加锁/解锁流程。

### 轻量级锁

轻量级锁是由偏向锁升级来的，偏向锁运行在一个线程进入同步块的情况下，当第二个线程加入锁竞争用的时候，偏向锁就会升级为轻量级锁；

### 重量级锁

重量锁在JVM中又叫对象监视器（Monitor），它很像C中的Mutex，除了具备Mutex(0|1)互斥的功能，它还负责实现了Semaphore(信号量)的功能，也就是说它至少包含一个竞争锁的队列，和一个信号阻塞队列（wait队列），前者负责做互斥，后一个用于做线程同步。

### 类锁、对象锁、重入锁

锁的持有者是线程而不是调用，所以不冲突，具有可重入性。

### wait、sleep的区别

wait释放锁，sleep继续持有锁

### Lock为什么性能大于synchronized

ReentrantLock底层是是volatile+CAS实现的，synchronized是悲观锁

### volatile原理

保证变量的可见性和顺序，是一种轻量级的互斥同步的实现

意义：
防止CPU指令重排，保证被volatile修饰的变量对所有线程都是可见的

如何保证可见性？
被volatile修饰的变量在工作内存修改后会被强制写回主内存，其他线程在使用时也会强制从主内存刷新，这样就保证了一致性。

### ReentranLock原理
![[ReenTrantLock原理#总结]]


## CopyOnWriteArrayList

在计算机中就是当你想要对一块内存进行修改时，我们不在原有内存块中进行写操作，而是将内存拷贝一份，在新的内存中进行写操作，写完之后呢，就将指向原来内存指针指向新的内存，原来的内存就可以被回收掉。

在添加这个数据的期间，其他线程如果要去读取数据，仍然是读取到旧的容器里的数据。

### 优点:

1. 据一致性完整，为什么？因为加锁了，并发数据不会乱。
2. 解决了像ArrayList、Vector这种集合多线程遍历迭代问题，记住，Vector虽然线程安全，只不过是加了synchronized关键字，迭代问题完全没有解决！
### 缺点:

1. 内存占有问题:很明显，两个数组同时驻扎在内存中，如果实际应用中，数据比较多，而且比较大的情况下，占用内存会比较大，针对这个其实可以用ConcurrentHashMap来代替。
2. 数据一致性:CopyOnWrite容器只能保证数据的最终一致性，不能保证数据的实时一致性。所以如果你希望写入的的数据，马上能读到，请不要使用CopyOnWrite容器。

### 使用场景

1. 读多写少（白名单，黑名单，商品类目的访问和更新场景），为什么？因为写的时候会复制新集合。
2. 集合不大，为什么？因为写的时候会复制新集合。
3. 实时性要求不高，为什么，因为有可能会读取到旧的集合数据。

## 什么是死锁？创造死锁的条件

AB两个线程由于互相持有对方需要的锁，而发生的阻塞现象，我们称为死锁；

四个条件：
- 互斥条件：一个资源每次只能被一个线程使用。
- 请求与保持条件：一个线程因请求资源而阻塞时，对已获得的资源保持不放。
- 不剥夺条件：线程已获得的资源，在未使用完之前，不能强行剥夺。
- 循环等待条件：若干线程之间形成一种头尾相接的循环等待资源关系。


## 多线程断点续传原理

在本地下载过程中要使用数据库实时存储到底存储到文件的哪个位置了，这样点击开始继续传递时，才能通过HTTP的GET请求中的setRequestProperty("Range","bytes=startIndex-endIndex");方法可以告诉服务器，数据从哪里开始，到哪里结束。同时在本地的文件写入时，RandomAccessFile的seek()方法也支持在文件中的任意位置进行写入操作。同时通过广播或事件总线机制将子线程的进度告诉Activity的进度条。关于断线续传的HTTP状态码是206，即HttpStatus.SC_PARTIAL_CONTENT。

## 如何安全停止一个线程

### 终止线程

1、使用violate boolean变量退出标志，使线程正常退出，也就是当run方法完成后线程终止。（推荐）

2、使用interrupt()方法中断线程，但是线程不一定会终止。

3、使用stop方法强行终止线程。不安全主要是：thread.stop()调用之后，创建子线程的线程就会抛出ThreadDeatherror的错误，并且会释放子线程所持有的所有锁。

### 终止线程池

ExecutorService线程池就提供了shutdown和shutdownNow这样的生命周期方法来关闭线程池自身以及它拥有的所有线程。

1、shutdown关闭线程池

线程池不会立刻退出，直到添加到线程池中的任务都已经处理完成，才会退出。

2、shutdownNow关闭线程池并中断任务

终止等待执行的线程，并返回它们的列表。试图停止所有正在执行的线程，试图终止的方法是调用Thread.interrupt()，但是大家知道，如果线程中没有sleep 、wait、Condition、定时锁等应用, interrupt()方法是无法中断当前的线程的。所以，ShutdownNow()并不代表线程池就一定立即就能退出，它可能必须要等待所有正在执行的任务都执行完成了才能退出。

# Android
## Service保活

- 在onStartCommend()将返回值设置为START_STICKY
- 在onDestroy()中重启
- 其他进程唤起
- 启动前台service
- 提高service优先级

Android 进程不死从3个层面入手：

### A.提供进程优先级，降低进程被杀死的概率
1. 监控手机锁屏解锁事件，在屏幕锁屏时启动1个像素的 Activity，在用户解锁时将 Activity 销毁掉。
2. 启动前台service。
3. 提升service优先级：

在AndroidManifest.xml文件中对于intent-filter可以通过android:priority = "1000"这个属性设置最高优先级，1000是最高值，如果数字越小则优先级越低，同时适用于广播。
### B. 在进程被杀死后，进行拉活
1. 注册高频率广播接收器，唤起进程。如网络变化，解锁屏幕，开机等
2. 双进程相互唤起。
3. 依靠系统唤起。
4. onDestroy方法里重启service：service + broadcast 方式，就是当service走ondestory的时候，发送一个自定义的广播，当收到广播的时候，重新启动service；
### C. 依靠第三方
根据终端不同，在小米手机（包括 MIUI）接入小米推送、华为手机接入华为推送；其他手机可以考虑接入腾讯信鸽或极光推送与小米推送做 A/B Test。

## 反编译

- apktools：反编译成原始目录，可以看到清单文件；
- dex2jar：将dex文件转化成一个classes.jar文件；
- jd-gui：将classes.jar转换为.java的源代码；

## Android中哪些用到了Binder

在Android系统中，很多组件和模块都使用了Binder作为进程间通信（IPC）方式。以下是一些使用了Binder的组件和模块：

1. Activity、Service、Broadcast、ContentProvider是Android系统中的四个主要组件，它们在多进程通信时底层都依赖于Binder IPC机制。例如，当进程A中的Activity要与进程B中的Service通信时，就需要依赖Binder IPC。
2. 调用系统服务，如获取输入法服务、闹钟服务、摄像头、电话等系统服务，都会用到进程间通信Binder。
3. 对于一些吃内存的模块，如地图模块、大图浏览、webview等，由于Android对每个进程的内存空间有限制，因此也使用了Binder机制进行进程间通信。

总之，Binder是Android系统中广泛使用的进程间通信机制，几乎所有的Android应用程序都会使用到它。


## Bitmap相关

[[图片压缩方案]]

### Bitmap 使用时候注意什么？

1、要选择合适的图片规格（bitmap类型）：

```Java
ALPHA_8   每个像素占用1byte内存        
ARGB_4444 每个像素占用2byte内存       
ARGB_8888 每个像素占用4byte内存（默认）      
RGB_565 每个像素占用2byte内存
```

2、降低采样率。BitmapFactory.Options 参数inSampleSize的使用，先把options.inJustDecodeBounds设为true，只是去读取图片的大小，在拿到图片的大小之后和要显示的大小做比较通过calculateInSampleSize()函数计算inSampleSize的具体值，得到值之后。options.inJustDecodeBounds设为false读图片资源。

3、复用内存。即，通过软引用(内存不够的时候才会回收掉)，复用内存块，不需要再重新给这个bitmap申请一块新的内存，避免了一次内存的分配和回收，从而改善了运行效率。

4、使用recycle()方法及时回收内存。

5、压缩图片。

## 为什么bindService可以跟Activity生命周期联动？

1. bindService 方法执行时，LoadedApk 会记录 ServiceConnection 信息。
2. Activity 执行 finish 方法时，会通过 LoadedApk 检查 Activity 是否存在未注销/解绑的 BroadcastReceiver 和 ServiceConnection，如果有，那么会通知 AMS 注销/解绑对应的 BroadcastReceiver 和 Service，并打印异常信息，告诉用户应该主动执行注销/解绑的操作。

## 性能优化相关

[[App稳定性优化]]

[[App绘制优化]]

[[App启动速度优化]]

[[App网络优化]]

[[App内存优化]]

[[App电量优化]]

[[App瘦身]]

[[App安全优化]]

## 稳定性优化

有哪些方面：
- Crash专项优化
- 性能稳定性优化
- 业务稳定性优化

#### 性能稳定性是怎么做的？

- 全面的性能优化：启动速度、内存优化、绘制优化
- 线下发现问题、优化为主
- 线上监控为主
- Crash专项优化

#### 业务稳定性如何保障？

- 数据采集+报警
- 主流程和核心业务埋点
- 异常监控+单点追查
- 兜底策略；

#### 如果发生了异常情况，怎么快速止损？

- 功能开关
- 统跳中心
- 动态修复：热修复、资源包更新
- 自主修复：安全模式

### 布局为什么会卡顿？怎么优化

#### 原因

1. xml是通过IO的方式加载到内存，IO过程可能会导致卡顿；
2. 布局加载过程是一个反射过程，反射过程也可能导致卡顿；
3. 层级较深，不合理的嵌套。
#### 优化方案

1. 异步inflate，AsyncLayoutInflater，它是通过在子线程对layout进行加载，加载完成后将View通过handler发送到主线程；
2. X2C框架，使用APT将布局文件转换为Java的方式，通过new对象来布局，省去了IO和反射带来的性能损耗；
3. 约束布局来写布局文件；
4. AOP和LayoutInflaterCompat.setFactory2来建立布局和控件加载速度的监控。

## 卡顿信息获取方案

Looper.loop()方法中有个Logging对象，它会在每个message处理前后都会被调用，主线程发生了卡顿，那就一定会在dispatchMessage方法中执行了耗时的代码，在执行一个message之前在子线程中postdelayed一个任务，时间为设定的阈值，如果message在规定时间内执行完成，就会取消掉这个任务，反之如果发生卡顿，那就会执行到这个任务，在这个任务中获取当前主线程执行的一个堆栈，那我们就可以知道哪里发生了卡顿。

高版本通过ANR-WatchDog获取权限；

## TextView setText耗时的原因，对TextView绘制层源码的理解

当textview的宽设置为wrap_content的时候，底层会调用checkForRelayout函数，这个函数根据文字的多少重新开始布局

因此将宽度设置为固定值或者match_parent的时候会大幅度减少绘制时间

## Apk瘦身

![[App瘦身]]

### 网络优化

1. 连接复用：keepAlive
2. 多个请求合并一个；
3. 减少请求数据大小：body用gzip压缩，http2.0可以header压缩
		（也可以考虑压缩返回的json数据的key数据的体积，尤其是针对返回数据格式变化不大的情况，支付宝聊天返回的数据用到了）。
4. 根据当前网络质量判断下载什么质量的图片；
5. 使用HttpDns优化DNS；绕过运营商的LocalDNS服务器，有效的防止了域名劫持，提高域名解析的效率。

## 电量优化

![[App电量优化]]


## 进程间通信会造成什么问题

- 静态成员和单例模式完全失效：独立的虚拟机造成。
- 线程同步机制完全失效：独立的虚拟机造成。
- SharedPreferences的可靠性下降：这是因为Sp不支持两个进程并发进行读写，有一定几率导致数据丢失。
- Application会多次创建：Android系统在创建新的进程时会分配独立的虚拟机，所以这个过程其实就是启动一个应用的过程，自然也会创建新的Application。

## Android中IPC方式及优缺点

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311062121530.png)


## 讲讲AIDL？如何优化多模块都使用AIDL的情况？

AIDL(Android Interface Definition Language，Android接口定义语言)：如果在一个进程中要调用另一个进程中对象的方法，可使用AIDL生成可序列化的参数，AIDL会生成一个服务端对象的代理类，通过它客户端可以实现间接调用服务端对象的方法。

AIDL的本质是系统提供了一套可快速实现Binder的工具。关键类和方法：

- AIDL接口：继承IInterface。
- Stub类：Binder的实现类，服务端通过这个类来提供服务。
- Proxy类：服务端的本地代理，客户端通过这个类调用服务端的方法。
- asInterface()：客户端调用，将服务端返回的Binder对象，转换成客户端所需要的AIDL接口类型的对象。如果客户端和服务端位于同一进程，则直接返回Stub对象本身，否则返回系统封装后的Stub.proxy对象。
- asBinder()：根据当前调用情况返回代理Proxy的Binder对象。
- onTransact()：运行在服务端的Binder线程池中，当客户端发起跨进程请求时，远程请求会通过系统底层封装后交由此方法来处理。
- transact()：运行在客户端，当客户端发起远程请求的同时将当前线程挂起。之后调用服务端的onTransact()直到远程请求返回，当前线程才继续执行。

当有多个业务模块都需要AIDL来进行IPC，此时需要为每个模块创建特定的aidl文件，那么相应的Service就会很多。必然会出现系统资源耗费严重、应用过度重量级的问题。解决办法是建立Binder连接池，即将每个业务模块的Binder请求统一转发到一个远程Service中去执行，从而避免重复创建Service。

工作原理：每个业务模块创建自己的AIDL接口并实现此接口，然后向服务端提供自己的唯一标识和其对应的Binder对象。服务端只需要一个Service并提供一个queryBinder接口，它会根据业务模块的特征来返回相应的Binder对象，不同的业务模块拿到所需的Binder对象后就可以进行远程方法的调用了。

[Binder连接池](https://codeleading.com/article/66891077957/)

[Android跨进程通信：图文详解 Binder机制 原理](https://blog.csdn.net/carson_ho/article/details/73560642)


### 跨进程传递大内存数据如何做？
Binder映射的最大内存只有 1024-8K，可以采用 binder + 匿名共享内存的形式，像跨进程传递大的 bitmap 需要打开系统底层的 ashmem 机制。

请按顺序仔细阅读下列文章提升对Binder机制的理解程度：

[写给 Android 应用工程师的 Binder 原理剖析](https://juejin.im/post/6844903589635162126 "https://juejin.im/post/6844903589635162126")

[Binder学习指南](https://link.juejin.cn/?target=http%3A%2F%2Fweishu.me%2F2016%2F01%2F12%2Fbinder-index-for-newer%2F "http://weishu.me/2016/01/12/binder-index-for-newer/")

[Binder设计与实现](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Funiversus%2Farticle%2Fdetails%2F6211589 "https://blog.csdn.net/universus/article/details/6211589")

[老罗Binder机制分析系列或Android系统源代码情景分析Binder章节](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fluoshengyang%2Farticle%2Fdetails%2F6618363 "https://blog.csdn.net/luoshengyang/article/details/6618363")

## 系统启动流程

init进程 -> Zygote进程 –> SystemServer进程 –> 各种系统服务 –> 应用进程

Android系统启动的核心流程如下：

1. 启动电源以及系统启动：当电源按下时引导芯片从预定义的地方（固化在ROM）开始执行，加载引导程序BootLoader到RAM，然后执行。
2. 引导程序BootLoader：BootLoader是在Android系统开始运行前的一个小程序，主要用于把系统OS拉起来并运行。
3. Linux内核启动：当内核启动时，设置缓存、被保护存储器、计划列表、加载驱动。当其完成系统设置时，会先在系统文件中寻找init.rc文件，并启动init进程。
4. init进程启动：初始化和启动属性服务，并且启动Zygote进程。
5. Zygote进程启动：创建JVM并为其注册JNI方法，创建服务器端Socket，启动SystemServer进程。
6. SystemServer进程启动：启动Binder线程池和SystemServiceManager，并且启动各种系统服务。
7. Launcher启动：被SystemServer进程启动的AMS会启动Launcher，Launcher启动后会将已安装应用的快捷图标显示到系统桌面上。


## 包安装过程

[[APK打包与安装]]


## [说下安卓虚拟机和java虚拟机的原理和不同点](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fjason0539%2Farticle%2Fdetails%2F50440669 "https://blog.csdn.net/jason0539/article/details/50440669")?（JVM、Davilk、ART三者的原理和区别）

JVM:.java -> javac -> .class -> jar -> .jar

架构: 堆和栈的架构.

DVM:.java -> javac -> .class -> dx.bat -> .dex

架构: 寄存器(cpu上的一块高速缓存)

### Android2个虚拟机的区别（一个5.0之前，一个5.0之后）

Dalvik是JIT编译器

ART是预编译字节码为机器语言，这一机制叫Ahead-Of-Time(AOT)编译，在安装的时候

执行更有效率，启动更快  

ART优点：

- 系统性能的显著提升。
- 应用启动更快、运行更快、体验更流畅、触感反馈更及时。
- 更长的电池续航能力。
- 支持更低的硬件。

ART缺点：

- 更大的存储空间占用，可能会增加10%-20%。
- 更长的应用安装时间。

## Jni方法互相调用

### Java调用C++

- Java中生命Native方法
- 通过Javac生成class文件，然后通过javah生成并导出.h头文件
- 实现头文件中的方法，编译成.so文件

### C++调用Java

- 查找ClassMethod这个类
- 获取类的构造方法ID
- 查找实例方法ID
- 创建该类的实例
- 调用对象的实例方法

## Jni中注册native的方式
### 静态注册

弊端：
1. 需要编译所有声明了native函数的Java类，每个所生成的class文件都得用javah命令生成一个头文件。
2. javah生成的JNI层函数名特别长，书写起来很不方便
3. 初次调用native函数时要根据函数名字搜索对应的JNI层函数来建立关联关系，这样会影响运行效率

### 动态注册

需要实现JNI_OnLoad方法

### 区别

静态注册
优点: 理解和使用方式简单, 属于傻瓜式操作, 使用相关工具按流程操作就行, 出错率低
缺点: 当需要更改类名,包名或者方法时, 需要按照之前方法重新生成头文件, 灵活性不高
动态注册
优点: 灵活性高, 更改类名,包名或方法时, 只需对更改模块进行少量修改, 效率高
缺点: 对新手来说稍微有点难理解, 同时会由于搞错签名, 方法, 导致注册失败


## Jni加载So方式

静态加载：
System.loadLibrary
动态加载：
System.load


## so 的加载流程是怎样的，生命周期是怎样的？

从 ClassLoader 的 PathList 中去找到目标路径加载的，同时 so 是通过 mmap 加载映射到虚拟空间的。生命周期加载库和卸载库时分别调用 JNI_OnLoad 和 JNI_OnUnload() 方法。


