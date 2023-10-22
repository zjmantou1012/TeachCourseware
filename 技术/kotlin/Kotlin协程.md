# 协程与线程的区别
线程是CPU调度最小单位
协程可以看作是一套轻量级的线程框架，将异步编程同步话，没有那么多的回调，协程内部自己维护了一套线程池，做了一些优化，比如无需关注线程切换，只需指定想要执行的线程。  

# 协程线程池的原理
1. 全局队列（阻塞+非阻塞）+ 本地队列
2. IO任务分发还有个缓存队列
3. 线程从队列里面寻找任务并执行，若使用IO分发器，则超出限制的任务将会放到缓存队列里面

## Dispatchers.Default原理
### 核心线程数8
### 适合密集计算任务

1. 创建任务加入到全局非阻塞队列里，并尝试唤醒空闲线程执行；
2. 如果失败了尝试创建新的线程执行；
3. 如果还是失败了就再次尝试唤醒空闲线程

![Dispatchers.Default原理](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309141917220.png)

## Dispatchers.IO原理
### 核心线程数64

1. 创建任务并加入到全局非阻塞队列，阻塞任务+1
2. 如果不跳过唤醒，则尝试唤醒空闲线程
3. 与Default类似

![Dispatchers.IO原理](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309141922558.png)


## 线程池调度
1. 只有获得cpu许可的线程才能执行计算型任务，而cpu许可的个数就是核心线程数
2. 如果线程没有找到可执行的任务，那么线程将会进入挂起状态，此时线程即为空闲状态
3. 当线程再次被唤醒后，会判断是否已经被终止，若是则退出，此时线程就销毁了

空闲状态到线程被唤醒：
- 线程挂起的时间到了
- 挂起过程中，有新的任务加入到线程池里，此时将会唤醒线程

# Java线程池与协程对比
- Java线程池开放了更多API，比较灵活
- 协程比较封闭，没有提供额外接口配置
- 可以通过系统参数来解决配置问题：
```kotlin
internal val CORE_POOL_SIZE = systemProp(
    //从这个属性里取值
    "kotlinx.coroutines.scheduler.core.pool.size",
    AVAILABLE_PROCESSORS.coerceAtLeast(2),//默认为cpu的个数
    minValue = CoroutineScheduler.MIN_SUPPORTED_POOL_SIZE//最小值为1
)

//修改Default属性值
    System.setProperty("kotlinx.coroutines.scheduler.core.pool.size", "20")

```

```kotlin
	//修改IO属性值
    System.setProperty("kotlinx.coroutines.io.parallelism", "40")
```

##### 参考链接
[来吧！接受Kotlin 协程--线程池的7个灵魂拷问](https://juejin.cn/post/7207078219215962170?searchId=2023091415590049A99B1AEBBE5322CD81)
