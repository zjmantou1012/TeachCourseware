---
author: zjmantou
title: 用Kotlin Flow解决Android开发中的痛点问题
time: 2024-04-02 周二
tags:
  - 资料
  - Android
  - MVI
---
> 大力智能技术团队-客户端 都梁人

## 前言

本文旨在通过实际业务场景阐述如何使用Kotlin Flow解决Android开发中的痛点问题，进而研究如何优雅地使用Flow以及纠正部分典型的使用误区。有关Flow的介绍及其操作符用法可以参考：[异步流 - Kotlin 语言中文站](https://link.juejin.cn/?target=https%3A%2F%2Fwww.kotlincn.net%2Fdocs%2Freference%2Fcoroutines%2Fflow.html "https://www.kotlincn.net/docs/reference/coroutines/flow.html")，本文不做赘述。基于LiveData+ViewModel的MVVM架构在某些场景下（以横竖屏为典型）存在局限性，本文会顺势介绍适合Android开发的基于Flow/Channel的MVI架构。

## 背景

大力智能客户端团队在平板端大力一起学App上深度适配了横竖屏场景，将原先基于Rxjava的MVP架构重构成基于LiveData+ViewModel+Kotlin协程的MVVM架构。随着业务场景的复杂度提升，LiveData作为数据的唯一载体似乎渐渐无法担此重任，其中一个痛点就是由于模糊了“状态”和“事件”的界限。LiveData的粘性机制会带来副作用，但这本身并不是LiveData的设计缺陷，而是对它的过度使用。

Kotlin Flow是基于kotlin协程的一套异步数据流框架，可以用于异步返回多个值。kotlin 1.4.0正式版发布时推出了StateFlow和SharedFlow，两者拥有Channel的很多特性，可以看作是将Flow推向台前，将Channel雪藏幕后的一手重要操作。对于新技术新框架，我们不会盲目接入，在经过调研试用一阶段后，发现Flow确实可以为业务开发止痛提效，下文分享这个探索的过程。

## 痛点一：蹩脚地处理ViewModel和View层通信

### 发现问题

#### 当屏幕可旋转后，LiveData不好用了？

项目由MVP过渡到MVVM时，其中一个典型的重构手段就是将Presenter中的回调写法改写成在ViewModel中持有LiveData由View层订阅，比如以下场景：

在大力自习室中，当老师切换至互动模式时，页面需要更改的同时还会弹出Toast提示模式已切换。

```kotlin
RoomViewModel.kt

class RoomViewModel : ViewModel() {

    private val _modeLiveData = MutableLiveData<Int>(-1)
    private val modeLiveData : LiveData<Int> = _mode

    fun switchMode(modeSpec : Int) {
        _modeLiveData.postValue(modeSpec)
    }
}
```

```kotlin
RoomActivity.kt

class RoomActivity : BaseActivity() {

    ...

    override fun initObserver() {
        roomViewModel.modeLiveData.observe(this, Observer {
            updateUI()
            showToast(it)
        })
    }
}
```

这样的写法乍一看没有毛病，但没有考虑到横竖屏切换如果伴随页面销毁重建的话，会导致在当前页面每次屏幕旋转都会重新执行observe，也就导致了每次旋转后都会弹一遍Toast。

LiveData会保证订阅者总能在值变化的时候观察到最新的值，并且每个初次订阅的观察者都会执行一次回调方法。这样的特性**对于维持** **UI** **和数据的一致性没有任何问题**，但想要观察LiveData来**发射一次性的事件就超出了其能力范围**。

当然，有一种解法通过保证LiveData同一个值只会触发一次onChanged回调，封装了MutableLiveData的**SingleLiveEvent**。先不谈它有没有其他问题，但就其对LiveData的魔改包装给我的第一感受是强扭的瓜不甜，违背了LiveData的设计思想，其次它就没有别的问题了吗？

#### ViewModel和View层的通信只依赖LiveData足够吗？

在使用MVVM架构时，数据变化驱动UI更新。对于UI来说只需关心最终状态，但对于一些事件，并不全是希望按照LiveData的合并策略将最新一条之前的事件全部丢弃。绝大部分情况是希望**每条事件都能被执行**，而LiveData并非为此设计。

在大力自习室中，老师会给表现好的同学点赞，收到点赞的同学会根据点赞类型弹出不同样式的点赞弹窗。为了防止横竖屏或者配置变化导致的重复弹窗，使用了上面提到的SingleLiveEvent

```kotlin
RoomViewModel.kt

class RoomViewModel : ViewModel() {

    private val praiseEvent = SingleLiveEvent<Int>()

    fun recvPraise(praiseType : Int) {
        praiseEvent.postValue(praiseType)
    }
}
```

```kotlin
RoomActivity.kt

class RoomActivity : BaseActivity() {

    ...

    override fun initObserver() {
        roomViewModel.praiseEvent.observe(this, Observer {
            showPraiseDialog(it)
        })
    }
}
```

考虑如下情况，老师同时给同学A“坐姿端正”和“互动积极”两种点赞，端上预期是要分别弹两次点赞弹窗。但根据上面的实现，如果两次recvPraise在一个UI刷新周期之内连续调用，即**liveData在很短的时间内连续post两次**，最终导致学生只会弹起第二个点赞的弹窗。

总的来说，上述两个问题根本都在于没有更好的手段去处理ViewModel和View层的通信，具体表现为对LiveData泛滥地使用以及没有对 **“状态”** 和 **“事件”** 进行区分

### 分析问题

根据上述总结，LiveData的确适合用来表示“状态”，但“事件”不应该是由某单个值表示。想要让View层顺序地消费每条事件，与此同时又不影响事件的发送，我的第一反应是使用一个阻塞队列来承载事件。但选型时我们要考虑以下问题，也是LiveData被推荐使用的优势 ：

1. 是否会发生内存泄漏，观察者的生命周期遭到销毁后能否自我清理
2. 是否支持线程切换，比如LiveData保证在主线程感知变化并更新UI
3. 不会在观察者非活跃状态下消费事件，比如LiveData防止因Activity停止时消费导致crash

#### **方案一：阻塞队列**

ViewModel持有阻塞队列，View层在主线程死循环读取队列内容。需要手动添加lifecycleObserver来保证线程的挂起和恢复，并且不支持协程。考虑使用kotlin协程中的Channel替代。

#### **方案二：** **Kotlin** **Channel**

Kotlin Channel和阻塞队列很类似，区别在于Channel用挂起的send操作代替了阻塞的put，用挂起的receive操作代替了阻塞的take。然后开启灵魂三问：

> 在生命周期组件中消费Channel是否会内存泄漏？

不会，因为Channel并不会持有生命周期组件的引用，并不像LiveData传入Observer式的使用。

> 是否支持线程切换？

支持，对Channel的收集需要开启协程，协程中可以切换协程上下文从而实现线程切换。

> 观察者非活跃状态下是否还会消费事件？

使用lifecycle-runtime-ktx库中的**launchWhenX**方法，对Channel的收集协程会在组件生命周期 < X时**挂起**，从而避免异常。也可以使用**repeatOnLifecycle(State)** 来在UI层收集，当生命周期 < State时，会**取消**协程，恢复时再重新启动协程。

看起来使用Channel承载事件是个不错的选择，并且一般来说事件分发都是一对一，因此并不需要支持一对多的BroadcastChannel（后者已经逐渐被废弃，被SharedFlow替代）

如何创建Channel？看一下Channel对外暴露可供使用的构造方法，考虑传入合适的参数。

```kotlin
public fun <E> Channel(

    // 缓冲区容量，当超出容量时会触发onBufferOverflow指定的策略
    capacity: Int = RENDEZVOUS,  

    // 缓冲区溢出策略，默认为挂起，还有DROP_OLDEST和DROP_LATEST
    onBufferOverflow: BufferOverflow = BufferOverflow.SUSPEND,

    // 处理元素未能成功送达处理的情况，如订阅者被取消或者抛异常
    onUndeliveredElement: ((E) -> Unit)? = null

): Channel<E>
```

首先Channel是热的，即任意时刻发送元素到Channel即使没有订阅者也会执行。所以考虑到存在订阅者协程被取消时发送事件的情况，即存在Channel处在无订阅者时的空档期收到事件情况。例如当Activity使用repeatOnLifecycle方法启动协程去消费ViewModel持有的Channel里的事件消息，当前Activity因为处于STOPED状态而取消了协程。

根据之前分析的诉求，空档期的事件不能丢弃，而应该在Activity回到活跃状态时依次消费。所以考虑当缓冲区溢出时策略为挂起，容量默认0即可，即**默认构造方法即符合我们的需求**。

之前我们提到，BroadcastChannel已经被SharedFlow替代，那我们用Flow代替Channel是否可行呢？

#### **方案三：普通Flow（冷流）**

Flow is cold, Channel is hot。所谓流是冷的即流的构造器中的代码直到流被收集时才会执行，下面是个非常经典的例子：

```kotlin
fun fibonacci(): Flow<BigInteger> = flow {
    var x = BigInteger.ZERO
    var y = BigInteger.ONE
    while (true) {
        emit(x)
        x = y.also {
            y += x
        }
    }
}

fibonacci().take(100).collect { println(it) } 
```

如果flow构造器里的代码不依赖订阅者独立执行，上面则会直接死循环，而实际运行发现是正常输出。

那么回到我们的问题，这里用冷流是否可行？显然并不合适，因为首先直观上冷流就无法在构造器以外发射数据。

但实际上答案并不绝对，通过在flow构造器内部使用channel，同样可以实现动态发射，如channelFlow。但是channelFlow本身不支持在构造器以外发射值，通过**Channel.receiveAsFlow**操作符可以将Channel转换成channelFlow。这样产生的Flow“外冷内热”，使用效果**和直接收集Channel几乎没有区别**。

```kotlin
private val testChannel: Channel<Int> = Channel()

private val testChannelFlow = testChannel.receiveAsFlow ()
```

#### **方案四：SharedFlow/StateFlow**

首先二者都是热流，并支持在构造器外发射数据。简单看下它们的构造方法

```kotlin
public fun <T> MutableSharedFlow(

    // 每个新的订阅者订阅时收到的回放的数目，默认0
    replay: Int = 0,

    // 除了replay数目之外，缓存的容量，默认0
    extraBufferCapacity: Int = 0,

    // 缓存区溢出时的策略，默认为挂起。只有当至少有一个订阅者时，onBufferOverflow才会生效。当无订阅者时，只有最近replay数目的值会保存，并且onBufferOverflow无效。 
    onBufferOverflow: BufferOverflow = BufferOverflow.SUSPEND
)
```

```kotlin
//MutableStateFlow等价于使用如下构造参数的SharedFlow

MutableSharedFlow(
    replay = 1,
    onBufferOverflow = BufferOverflow.DROP_OLDEST
)
```

SharedFlow被Pass的原因主要有两个：

1. SharedFlow支持被多个订阅者订阅，导致同一个事件会被多次消费，并不符合预期。
2. 如果认为1还可以通过开发规范控制，SharedFlow的在**无订阅者时会丢弃数据**的特性则让其彻底无缘被选用承载必须被执行的事件

而StateFlow可以理解成特殊的SharedFlow，也就无论如何都会有上面两点问题。

当然，适合使用SharedFlow/StateFlow的场景也有很多，下文还会重点研究。

#### 总结

对于想要在ViewModel层发射**必须执行且只能执行一次的事件**让View层执行时，不要再通过向LiveData postValue让View层监听实现。推荐使用Channel或者是通过Channel.receiveAsFlow方法创建的ChannelFlow来实现ViewModel层的事件发送。

### 解决问题

```kotlin
RoomViewModel.kt

class RoomViewModel : ViewModel() {

    private val _effect = Channel<Effect> = Channel ()
    val effect = _effect. receiveAsFlow ()

    private fun setEffect(builder: () -> Effect) {
        val newEffect = builder()
        viewModelScope.launch {
            _effect.send(newEffect)
        }
    }

    fun showToast(text : String) {
        setEffect {
            Effect.ShowToastEffect(text)
        }
    }
}

sealed class Effect {
    data class ShowToastEffect(val text: String) : Effect()
}
```

```kotlin
RoomActivity.kt

class RoomActivity : BaseActivity() {

    ...

    override fun initObserver() {
        lifecycleScope.launchWhenStarted {
            viewModel.effect.collect {
                when (it) {
                    is Effect.ShowToastEffect -> {
                        showToast(it.text)
                    }
                }
            }
       }
    }
}
```

## 痛点二：Activity/Fragment通过共享ViewModel通信的问题

我们经常让Activity和其中的Fragment共同持有由Acitivity作为ViewModelStoreOwner构造的ViewModel，来实现Activity和Fragment、以及Fragment之间的通信。典型场景如下：

```kotlin
class MyActivity : BaseActivity() {

    private val viewModel : MyViewModel by viewModels()

    private fun initObserver() {
        viewModel.countLiveData.observe { it->
            updateUI(it)
        }
    }

    

    private fun initListener() {
        button.setOnClickListener {
            viewModel.increaseCount()
        }
    }
}

class MyFragment : BaseFragment() {

    private val activityVM : MyViewModel by activityViewModels()  

    private fun initObserver() {
        activityVM.countLiveData.observe { it->
            updateUI(it)
        }
    }
}

class MyViewModel : ViewModel() {   

    private val _countLiveData = MutableLiveData<Int>(0)

    private val countLiveData : LiveData<Int> = _countLiveData

    fun increaseCount() {
        _countLiveData.value = 1 + _countLiveData.value ?: 0
    }

}
```

简单来说就是通过让Activity和Fragment观察同一个liveData，实现一致性。

那如果是要在Fragment中调用Activity的方法，通过共享ViewModel可行吗？

### 发现问题

#### DialogFragment和Activity的通信

我们通常使用DialogFragment来实现弹窗，在其宿主Activity中设置弹窗的点击事件时，如果回调函数中引用了Activity对象，则很容易产生由横竖屏页面重建引发的引用错误。所以我们建议让Activity实现接口，在弹窗每次Attach时都会将当前附着的Activity强转成接口对象来设置回调方法。

```kotlin
class NoticeDialogFragment : DialogFragment() {

    internal lateinit var listener: NoticeDialogListener    

    interface NoticeDialogListener {
        fun onDialogPositiveClick(dialog: DialogFragment)
        fun onDialogNegativeClick(dialog: DialogFragment)
    }

    override fun onAttach(context: Context) {
        super.onAttach(context)
        try {
            listener = context as NoticeDialogListener
        } catch (e: ClassCastException) {
            throw ClassCastException((context.toString() +
                    " must implement NoticeDialogListener"))
        }
    }
}
```

```kotlin
class MainActivity : FragmentActivity(), NoticeDialogFragment.NoticeDialogListener {

    fun showNoticeDialog() {
        val dialog = NoticeDialogFragment()
        dialog.show(supportFragmentManager, "NoticeDialogFragment")
    }

    override fun onDialogPositiveClick(dialog: DialogFragment) {
        // User touched the dialog's positive button
    }

    override fun onDialogNegativeClick(dialog: DialogFragment) {
        // User touched the dialog's negative button
    }

}
```

这样的写法不会有上述问题，但是随着页面上支持的弹窗变多，Activity需要实现的接口也越来越多，无论是对编码还是阅读代码都不是很友好。那有没有机会借用共享的ViewModel做点文章？

### 分析问题

我们想要向ViewModel发送事件，并让所有依赖它的组件接收到事件。比如在FragmentA点击按键触发事件A，其宿主Activity、相同宿主的FragmentB和FragmentA其本身都需要响应该事件。

有点像广播，且具有两个特性：

1. 支持一对多，即一条消息支持被多个订阅者消费
2. 具有时效性，过期的消息没有意义且不应该被延迟消费。

看起来EventBus是一种实现方法，但是已经有了ViewModel作为媒介再使用显然有些浪费，EventBus还是更适合跨页面、跨组件的通信。对比前面分析的几种模型的使用，发现SharedFlow在这个场景下非常有用武之地。

1. SharedFlow类似BroadcastChannel，支持多个订阅者，一次发送多处消费。
2. SharedFlow配置灵活，如默认配置 capacity = 0， replay = 0，意味着新订阅者不会收到类似LiveData的回放。无订阅者时会直接丢弃，正符合上述时效性事件的特点。

### 解决问题

```kotlin
class NoticeDialogFragment : DialogFragment() {

    private val activityVM : MyViewModel by activityViewModels()

    fun initListener() {
        posBtn.setOnClickListener {
            activityVM.sendEvent(NoticeDialogPosClickEvent(textField.text))
            dismiss()
        }

        negBtn.setOnClickListener {
            activityVM.sendEvent(NoticeDialogNegClickEvent)
            dismiss()
        }
    }
}

class MainActivity : FragmentActivity() {

    private val viewModel : MyViewModel by viewModels()
    

    fun showNoticeDialog() {
        val dialog = NoticeDialogFragment()
        dialog.show(supportFragmentManager, "NoticeDialogFragment")
    }

    fun initObserver() {
        lifecycleScope.launchWhenStarted {
           viewModel.event.collect {
                when(it) {
                    is NoticeDialogPosClickEvent -> {
                        handleNoticePosClicked(it.text)
                    }

                    NoticeDialogNegClickEvent -> {
                        handleNoticeNegClicked()
                    }
                }
            }
        }
    }
}

class MyViewModel : ViewModel() {

    private val _event: MutableSharedFlow<Event> = MutableSharedFlow ()
    

    val event = _event. asSharedFlow ()

    fun sendEvent(event: Event) {
        viewModelScope.launch {
            _event.emit(event)
        }
    }
}
```

这里通过_lifecycleScope.launchWhenX_启动协程其实并不是最佳实践，如果想要Activity在非活跃状态下直接丢弃收到的事件，应该使用repeatOnLifecycle来控制协程的开启和取消而非挂起。但考虑到DialogFragment的存活周期是宿主Activity的子集，所以这里没有大问题。

## 基于Flow/Channel的MVI架构

前面讲的痛点问题，实际上是为了接下来要介绍的MVI架构抛砖引玉。而MVI架构的具体实现，也就是将上述解决方案融合到模版代码中，最大程度发挥架构的优势。

### MVI是什么

所谓MVI，对应的分别是Model、View、Intent

Model： 不是MVC、MVP里M所代指的数据层，而是指**表征** **UI** **状态的聚合对象**。Model是不可变的，Model与呈现出的UI是一一对应的关系。

View：和MVC、MVP里做代指的V一样，指渲染UI的单元，可以是Activity或者View。可以接收用户的交互意图，会根据新的Model响应式地绘制UI。

Intent：不是传统的Android设计里的Intent，一般指用户与UI交互的意图，如按钮点击。**Intent是改变Model的唯一来源**。

三者的关系可以用下面这张图来表示，MVI的模型更强调单向数据流动（V -> I -> M -> V）和唯一数据源Model

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c3190b28151d4c6bb06018ae09b9dd0f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

但图里的模型太过理想化，实际开发中intent并不完全来自于用户与UI的交互，还会来自后台任务比如消息服务等。intent除了可以通过改变Model来更新UI外，还会伴随着一些副作用如弹Toast等。但这并不影响抽象出的最小模型里的单向数据流的思想。

对比MVVM的区别主要在哪？

1. MVVM并没有约束View层与ViewModel的交互方式，具体来说就是View层可以随意调用ViewModel中的方法，而MVI架构下ViewModel的实现对View层屏蔽，只能通过发送Intent来驱动事件。
2. MVVM架构并不强调对表征UI状态的Model值收敛，并且对能影响UI的值的修改可以散布在各个可被直接调用的方法内部。而MVI架构下，Intent是驱动UI变化的唯一来源，并且表征UI状态的值收敛在一个变量里。

### 基于Flow/Channel的MVI如何实现

抽象出基类BaseViewModel

UiState是可以表征UI的Model，用StateFlow承载（也可以使用LiveData）

UiEvent是表示交互事件的Intent，用SharedFlow承载

UiEffect是事件带来除了改变UI以外的副作用，用channelFlow承载

```kotlin
BaseViewModel.kt

abstract class BaseViewModel<State : UiState, Event : UiEvent, Effect : UiEffect> : ViewModel() {

    /**
     * 初始状态
     * stateFlow区别于LiveData必须有初始值
     */
    private val initialState: State by lazy { createInitialState() }

    abstract fun createInitialState(): State

    /**
     * uiState聚合页面的全部UI 状态
     */
    private val _uiState: MutableStateFlow<State> = MutableStateFlow(initialState)
    

    val uiState = _uiState.asStateFlow()

    /**
     * event包含用户与ui的交互（如点击操作），也有来自后台的消息（如切换自习模式）
     */
     private val _event: MutableSharedFlow<Event> = MutableSharedFlow()
     

     val event = _event.asSharedFlow()
     

     

    /**
     * effect用作 事件带来的副作用，通常是 一次性事件 且 一对一的订阅关系
     * 例如：弹Toast、导航Fragment等
     */
     private val _effect: Channel<Effect> = Channel()

     val effect = _effect.receiveAsFlow()

    init {
        subscribeEvents()
    }

    private fun subscribeEvents() {
        viewModelScope.launch {
            event.collect {
                handleEvent(it)
            }
        }
    }

    protected abstract fun handleEvent(event: Event)

    fun sendEvent(event: Event) {
        viewModelScope.launch {
            _event.emit(event)
        }
     }

    protected fun setState(reduce: State.() -> State) {
        val newState = currentState.reduce()
        _uiState.value = newState
    }

    protected fun setEffect(builder: () -> Effect) {
        val newEffect = builder()
        viewModelScope.launch {
            _effect.send(newEffect)
        }
     }
}

interface UiState

interface UiEvent

interface UiEffect
```

StateFlow基本等同于LiveData，区别在于StateFlow必须有初值，这也更符合页面必须有初始状态的逻辑。一般使用data class实现UiState，页面所有元素的状态用成员变量表示。

用户交互事件用SharedFlow，具有时效性且支持一对多订阅，使用它可以解决上文提到的痛点二问题。

消费事件带来的副作用影响用ChannelFlow承载，不会丢失且一对一订阅，只执行一次。使用它可以解决上文提到的痛点一问题。

协议类，定义具体业务需要的State、Event、Effect类

```kotlin
class NoteContract {

    /**
    * pageTitle: 页面标题
    * loadStatus: 上拉加载的状态
    * refreshStatus: 下拉刷新的状态
    * noteList : 备忘录列表
    */
    data class State(
        val pageTitle: String,
        val loadStatus: LoadStatus,
        val refreshStatus: RefreshStatus,
        val noteList: MutableList<NoteItem>
    ) : UiState

    sealed class Event : UiEvent {
        // 下拉刷新事件
        object RefreshNoteListEvent : Event()
        

        // 上拉加载事件
        object LoadMoreNoteListEvent: Event()

        // 添加按键点击事件
        object AddingButtonClickEvent : Event()

        // 列表item点击事件
        data class ListItemClickEvent(val item: NoteItem) : Event()

        // 添加项弹窗消失事件
        object AddingNoteDialogDismiss : Event()

        // 添加项弹窗添加确认点击事件
        data class AddingNoteDialogConfirm(val title: String, val desc: String) : Event()

        // 添加项弹窗取消确认点击事件
        object AddingNoteDialogCanceled : Event()
    }

    sealed class Effect : UiEffect {
    

        // 弹出数据加载错误Toast
        data class ShowErrorToastEffect(val text: String) : Effect()

        // 弹出添加项弹窗
        object ShowAddNoteDialog : Effect()
    }

    sealed class LoadStatus {

        object LoadMoreInit : LoadStatus()

        object LoadMoreLoading : LoadStatus()

        data class LoadMoreSuccess(val hasMore: Boolean) : LoadStatus()

        data class LoadMoreError(val exception: Throwable) : LoadStatus()

        data class LoadMoreFailed(val errCode: Int) : LoadStatus()

    }

    sealed class RefreshStatus {

        object RefreshInit : RefreshStatus()

        object RefreshLoading : RefreshStatus()

        data class RefreshSuccess(val hasMore: Boolean) : RefreshStatus()

        data class RefreshError(val exception: Throwable) : RefreshStatus()

        data class RefreshFailed(val errCode: Int) : RefreshStatus()

    }

}
```

在生命周期组件中收集状态变化流和一次性事件流，发送用户交互事件

```kotlin
class NotePadActivity : BaseActivity() {

      ...

    override fun initObserver() {
        super.initObserver()
        lifecycleScope.launchWhenStarted {
            viewModel.uiState.collect {
                when (it.loadStatus) {
                    is NoteContract.LoadStatus.LoadMoreLoading -> {
                        adapter.loadMoreModule.loadMoreToLoading()
                    }
                    ...
                }
                

                when (it.refreshStatus) {
                    is NoteContract.RefreshStatus.RefreshSuccess -> {
                        adapter.setDiffNewData(it.noteList)
                        refresh_layout.finishRefresh()
                        if (it.refreshStatus.hasMore) {
                            adapter.loadMoreModule.loadMoreComplete()
                        } else {
                            adapter.loadMoreModule.loadMoreEnd(false)
                        }
                    }
                    ...
                }
                

                txv_title.text = it.pageTitle
                txv_desc.text = "${it.noteList.size}条记录"
            }
        }

        lifecycleScope.launchWhenStarted {
            viewModel.effect.collect {
                when (it) {
                

                    is NoteContract.Effect.ShowErrorToastEffect -> {
                        showToast(it.text)
                    }
                    

                    is NoteContract.Effect.ShowAddNoteDialog -> {
                        showAddNoteDialog()
                    }
                }
            }
        }
    }

    private fun initListener() {
        btn_floating.setOnClickListener {
            viewModel.sendEvent(NoteContract.Event.AddingButtonClickEvent)
        }
    }

}
```

### 使用MVI有哪些好处

1. 解决了上文的两个痛点。这也是我花很长的篇幅去介绍解决两个问题过程的原因。只有真的痛过才会感受到选择合适架构的优势。
2. 单向数据流，任何状态的变化都来自事件，因此更容易定位出问题。
3. 理想情况下对View层和ViewModel层做了接口隔离，更加解耦。
4. 状态、事件从架构层面上就明确划分，便于约束开发者写出漂亮的代码。

### 实际使用下来的问题

1. 膨胀的UiState，当页面复杂度提高，表示UiState的data class会严重膨胀，并且由于其牵一发而动全身的特点，想要局部更新的代价很大。因此对于复杂页面，可以通过拆分模块，让各个Fragment/View分别持有各自的ViewModel来拆解复杂度。
2. 对于大部分的事件处理都只是调用方法，相比直接调用额外多了定义事件类型和中转部分的编码。

### 结论

架构中对SharedFlow和channelFlow的使用绝对值得保留，就算不使用MVI架构，参考这里的实现也可以帮助解决很多开发中的难题，尤其是涉及横竖屏的问题。

可以选择使用StateFlow/LiveData收敛页面全部状态，也可以拆分成多个。但更加建议按UI组件模块拆分收敛。

跳过使用Intent，直接调用ViewModel方法也可以接受。

## 使用Flow还能给我们带来什么

**比Rxjava更简单，比LiveData更多的操作符**

如使用flowOn操作符切换协程上下文、使用buffer、conflate操作符处理背压、使用debounce操作符实现防抖、使用combine操作符实现flow的组合等等。

**比直接使用协程更简单地将基于回调的api改写成像同步代码一样的调用**

使用callbackFlow，将异步操作结果以同步挂起的形式发射出去。

## 结语

我认为每当推出新技术，我们不一定就要立即投入学习使用，再牛的框架脱离了业务场景也只是空中楼阁。但如果新技术真的能够解决我们开发中的痛点，理解并付诸实践一下或许能开阔你的思路，正如当你也遇到了和文章中相似的问题，试试使用Flow吧！


# [原文链接](https://juejin.cn/post/7031726493906829319)
