---
author: zjmantou
title: 序列化-Serializable和Parcelable
time: 2023-10-24 周二
tags:
  - Android
  - 技术
---
Parcelable适合Binder通信用，开销小，速度快；
Serializable适合做数据持久化；因为开销大；
	因为使用了反射机制，会有大量的临时变量，可能会导致频繁的GC；写数据的过程中通过IO流的形式。

### 相关面试题

#### 1.Android里面为什么要设计出Bundle而不是直接用Map结构
1. Bundle内部是由ArrayMap实现的，ArrayMap的内部实现是两个数组，一个int数组是存储对象数据对应下标，一个对象数组保存key和value，内部使用二分法对key进行排序，所以在添加、删除、查找数据的时候，都会使用二分法查找，只适合于小数据量操作，如果在数据量比较大的情况下，那么它的性能将退化。而HashMap内部则是数组+链表结构，所以在数据量较少的时候，HashMap的Entry Array比ArrayMap占用更多的内存。因为使用Bundle的场景大多数为小数据量，我没见过在两个Activity之间传递10个以上数据的场景，所以相比之下，在这种情况下使用ArrayMap保存数据，在操作速度和内存占用上都具有优势，因此使用Bundle来传递数据，可以保证更快的速度和更少的内存占用。

2. 另外一个原因，则是在Android中如果使用Intent来携带数据的话，需要数据是基本类型或者是可序列化类型，HashMap使用Serializable进行序列化，而Bundle则是使用Parcelable进行序列化。而在Android平台中，更推荐使用Parcelable实现序列化，虽然写法复杂，但是开销更小，所以为了更加快速的进行数据的序列化和反序列化，系统封装了Bundle类，方便我们进行数据的传输。

#### 2.Android中Intent/Bundle的通信原理及大小限制
Intent 中的 Bundle 是使用 Binder 机制进行数据传送的。能使用的 Binder 的缓冲区是有大小限制的（有些手机是 2 M），而一个进程默认有 16 个 Binder 线程，所以一个线程能占用的缓冲区就更小了（ 有人以前做过测试，大约一个线程可以占用 128 KB）。所以当你看到 The Bindertransaction failed because it was too large 这类 TransactionTooLargeException 异常时，你应该知道怎么解决了

#### 3.为何Intent不能直接在组件间传递对象而要通过序列化机制？
Intent在启动其他组件时，会离开当前应用程序进程，进入ActivityManagerService进程（intent.prepareToLeaveProcess()），这也就意味着，Intent所携带的数据要能够在不同进程间传输。首先我们知道，Android是基于Linux系统，不同进程之间的java对象是无法传输，所以我们此处要对对象进行序列化，从而实现对象在 应用程序进程 和 ActivityManagerService进程 之间
传输。而Parcel或者Serializable都可以将对象序列化，其中，Serializable使用方便，但性能不如Parcel容器，后者也是Android系统专门推出的用于进程间通信等的接口

作者：任振铭
链接：https://www.jianshu.com/p/6a84f63ea7bd
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。