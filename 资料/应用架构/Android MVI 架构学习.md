---
author: zjmantou
title: Android MVI 架构学习
time: 2024-04-02 周二
tags:
  - 资料
  - Android
  - MVI
---
## 1. 概述

### [](#)[](#)1.1 Android 架构的背景

这些年来，Android 上发展了多种主流架构，从最开始的 `MVC`，到 `Clean` 和 `MVP`，再到现在最火热的 `MVVM`，可以说架构发展一直很卷，这不，`MVVM` 还没有用个几年呢， `MVI` 就出来了。

要说 Android 架构卷，实则不然，上面说的这些架构其实根本不是来自 Android 的，而是源自于 **Web，即大前端**， Web 由于其自身特性（还不算完全成熟，业务多且杂，热部署等），版本迭代速度巨快，技术的更新迭代也因此很快，上面这些架构最早就是在前端所应用和发展出来的， 而移动端则是直接抄来，跟着Web的步伐前进。

所以 `MVI` 显然不是 Android 架构的终点，说不定明年 Web 上就又弄出一个新玩意把它取代了。不学习 `MVI` 并不会让我们落伍，现阶段 `MVP`、`MVVM` 足以解决Android所有的业务场景。

但是学习 `MVI` 有这么几个好处：

1. **装杯**：  
    新鲜的架构会让代码逼格提升，展示代码时，你可以用 “唯一可信刷新点”、“数据单向流动” 等词语来装杯，而且 MVI 会使用协程`Flow`，Flow 可是 Google 这两年很推荐的框架呢
2. **官方认定**：  
    Android 官网在去年就已经有推荐使用 `MVI` 的文章了， 在 Google 开发者大会中也有专门直播介绍 MVI 架构，所以它是官方认可的架构，至少不会那么容易被淘汰，而且面试也有可能会问到
3. **素材好找，容易引用**：  
    `MVI` 并不会引入什么三方库。比起具体的架构形态，它的形态更像是一种设计思想，基于已有的 `MVP`、`MVVM` 进行调整，也能搞出 `MVI`，所以它其实离我们很近，方便我们直接拿代码出来撸

除此之外就真真真真没了，各种文章都会踩一捧一说 `MVI` 的多个优点（虽然本文也会一样），但是架构终究是为项目服务，如果你的项目能够快速开发出一个 `MVP` 的界面，你就可以花更多时间在 Debug、单元测试等提升质量的事情上， 而如果你的项目比较慢才能搞出一个 `MVVM` / `MVI` 页面，那因为时间问题，你很有可能就少测几个用例，多埋了几个雷。

So ， `MVI` 目前并不在 Android 的必修课中，你不会也不用烦恼，请抱着休闲的心态来学习吧~

### [](#)[](#)1.2 MVC

MVC 是最早的明确把 Android 页面框架划分为 **视图层（View）、逻辑层（Controller）和 数据模型层（Model）** 的架构，它们的关系如下图所示：  
![在这里插入图片描述](file:///Users/mantou/.config/joplin-desktop/resources/250c7e0e4cd14c22bd40ed2919461218.png)  
逻辑单元的流动过程是：

- View层 可以调用 Controller层，触发一些逻辑操作，例如点击视图某个按钮，可以调用网络请求，也可以直接修改 Model层数据
- Controller层 操作后，可以直接操作 Model层，例如对数据库、内存的修改
- View层 直接持有 Model层，所以在感知其变化后，会触发 UI 更新，以将最新的数据展示在屏幕上

MVC 的缺点有：

- Controller层 往往都已 `Activity` / `Fragment` 为载体，它们正好又是视图UI，所以页面逻辑稍微复杂一点，就会导致 `Activity` 的代码臃肿膨胀，不好维护，代码可读性差
- View层 和 Model层 直接耦合，虽然看起来很方便，但是这会出现一个重要问题：**可复用性、扩展能力差**  
    ①：对于 Model 来说：假设网络接口调整，一个数据 Bean 的结构需要修改，它除了自身代码改变，它还会直接影响到 View 层的代码，触一发而动全身，扩展性很差  
    ②：对于 View 来说：因为和 Model 进行直接绑定，所以不好在其它数据源不同的地方复用
- Model层 会被多个地方修改，出现问题时，得从 View层 和 Controller层 进行 Debug

### [](#)[](#)1.3 MVP

MVP 的好处就是把 MVC 的缺点解决了， View层 和 Model层 不直接耦合，将逻辑层改了个名字，叫 `Presenter`，如下图所示：  
![在这里插入图片描述](file:///Users/mantou/.config/joplin-desktop/resources/8c309d5e19b94d5092a67d71926f0b89.png)  
逻辑单元的流动过程是：

- View 直接持有 Presenter，可以通知 Presenter 进行逻辑操作
- Presenter 可以通过网络请求等数据处理逻辑，操作 Model
- Model 通过回调或其它方式通知 Presenter
- Presenter 通过**接口**或**直接持有 View** 来通知 View 进行 UI更新

MVP 的缺点是：

- 因为 Presenter 会以接口的方式来通知 View 更新 UI，所以复杂业务容易导致接口函数爆炸多，不符合**接口的方法应该尽可能少**的原则
- Presenter 无法感知 View 的生命周期，比如在 View 销毁后，可能 Presenter 还在做一些耗时操作，导致出现性能问题

### [](#)[](#)1.4 MVVM（无 DataBinding 版）

Jetpack 出来后，它通过 `LifeCycle` 为 Presenter 赋予了能感知 View 生命周期的能力，并改了个名字叫 `ViewModel`。

Jetpack 甚至直接提供了 `ViewModel` 类、 `LiveData` 类等来让我们使用，非常方便。

```kotlin
class MyViewModel : ViewModel() {
    override fun onCleared() {
        // 释放资源
    }
}

```

在没有使用 `DataBinding` 时， MVVM 和 MVP 其实是差不多的，如下图所示：  
![在这里插入图片描述](file:///Users/mantou/.config/joplin-desktop/resources/24d41b6b02ee4fafbedc8b221c7c9887.png)

无 DataBinding 版的缺点是：

- MVVM 的初衷是数据双向绑定， 无 `DataBinding` 版的 MVVM 没有做到这点
- View 会同时订阅多个 `LiveData`， 每个 `LiveData` 都可以看做是触发 View 更新的一个刷新点，假设 UI 展示出现异常，我们需要从众多刷新点中找到有问题的那一个，调试上可能会比较麻烦~

### [](#)[](#)1.5 MVVM（DataBinding 版）

**MVVM的核心是双向绑定**，也就是 View 和 ViewModel 的一种自动绑定，这种能力是：

- `View` 可以直接触达 `ViewModel` 逻辑，而无需通过在代码中写 `setClickListener{ viewModel.onClick() }` 等逻辑
- `ViewModel` 的变化可以直接更新 `View`， 而无需通过在 `View` 代码中写 `setText=xxx`、`setXXX` 等逻辑

总结就是加了 `DataBinding` / `ViewBinding` 后 ，好处就是 `Activity` / `Fragment` 可以少写一些诸如 `findViewById`、`setXXX` 等的样板代码：  
![在这里插入图片描述](file:///Users/mantou/.config/joplin-desktop/resources/c417a2777b9e41feaf49ca822efa7279.png)

MVVM 有 DataBinding 版的缺点是：

- 双向绑定对调试不太友好，UI 出现异常时，需要定位问题是出现在 V 层还是出现在 M 层，虽然 MVP 也是这样，但是用了 `DataBinding` 的代码，往往都很不直观，跳来跳去的看得费神
- `DataBinding` 会直接注入到 View 的 XML 文件中，这使得这个 View 不好在其它地方被直接复用

### [](#)[](#)1.6 MVI 的起源

MVI 模式来源于2014年的 `Cycle.js`(一个 JavaScript框架)，并且在主流的 JS 框架 `Redux` 中大行其道，然后就被一些大佬移植到了 Android 上（比如最早期用Java写的 [mosby](https://github.com/sockeqwe/mosby "https://github.com/sockeqwe/mosby")）。

在 [MVI(Model-View-Intent) Pattern in Android](https://medium.com/code-yoga/mvi-model-view-intent-pattern-in-android-98c143d1ee7c "https://medium.com/code-yoga/mvi-model-view-intent-pattern-in-android-98c143d1ee7c")这篇文章上说明了搞出 MVI 是为了解决什么问题的：

> **对于主要的GUI体系结构，MVI的定义相当松散，它的核心是回归MVC提供的单向数据流…… 尽管还有一些其他的部分**

单看这句话，MVI 就像是 MVC 的一种衍生产物， 我们知道 MVP 也是 MVC 的衍生产物，MVVM 也是 MVP 的一种衍生，如下图所示（因为 MVI 是一种松散的定义，而 MVP、MVVM是一种对框架的强制定义，所以我对 MVI 使用了虚线）：  
![在这里插入图片描述](file:///Users/mantou/.config/joplin-desktop/resources/740ac067d9604a9b991f8828deb8ebb7.png)

这样看来，MVI 好像不是根据最先进的 MVVM 发展而来的，而是一种新分支，**它的出现不是为了解决 MVP、MVVM 所存在的问题，而是基于MVC提供的M和V层来实现一些能力。**

下面，我们将来探究 MVI 具备样什么特性。

## [](#)[](#)2. MVI 特性

MVI 的核心是：**唯一可信数据源**的**单向流动**。

### [](#)[](#)2.1 数据的单向流动

数据的单向流动，其实就是不想让数据双向流动。

数据的双向流动就是使用 `DataBinding` 那样：数据模型的变化会同步到视图上，视图上的操作同时也会同步到数据模型上。

而数据的单向流动，强调的是**数据的源头只有一个，目的地只有一个，数据的流向是易追踪的**。

这两者其实可以下面这种简单的方式来区分：

- 如果需要手动让 `View` 观察 `ViewModel`（视图数据）来更新自身，那么数据就是单向流动的
- 如果不需要手动让 `View` 观察， `ViewModel` 的更新能够自动触发 `View` 的更新，那么数据就是双向绑定的

那其实 MVC、MVP、MVVM（无 `DataBinding`） 都是数据单向流动的框架。

### [](#)[](#)2.2 唯一可信数据源

MVI 的愿景是**能让 View 触发刷新的状态只有一个**。

举个例子，假设一个 View 上有多个 UI 控件，用户不同的操作可以触发不同的 UI 控件刷新：
```kotlin
// Activity  /  Fragment 的代码
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        vm = viewModel()
        
        // 是否弹出/关闭 Loading 的动画
        vm.showLoading.observe(this, Observer {
            showOrCloseLoading(it)
        })
        // button 是否要置灰
        vm.buttonState.observe(this, Observer {
            setButtonState(it)
        })
        // textView 更新UI
        vm.titleText.observe(this, Observer {
            textView.text = it.toString()
        })
        ... 设置一些监听用户操作的事件
    }

```

上面代码中有三个可以影响 UI 刷新的地方，也就是说有三个更新点。

而 MVI 中**只能有一个更新点**， 上面的代码要做到只有一个更新点，那就相当于要收归所有更新的地方，变成下面这样：

```kotlin
sealed class UiState

class LoadingState(showLoading: Boolean): UiState()
class ButtonState(color: Color): UiState()
class TitleState(title: String): UiState()

...

vm.uiState.observe(this, Observer { state ->
    when (state) {
        is LoadingState -> showOrCloseLoading(state.value),
        is ButtonState -> setButtonState(state.color)
        is TitleState -> textView.text = state.title
    }
})

```

虽然第一眼看上去没有什么卵用，但上面代码确实做到了**唯一可信数据源**，`uiState` 是数据源，而 UI 刷新只依赖这个数据源，就让它具备了唯一可信的属性。（因为不会在有别的状态会触发 UI 更新了！）

### [](#)[](#)2.3 MVI 各层

MVI 将架构分成了三个部分：

- `I - Intent` **意图**：它是**简单描述用户与App交互时产生的一个动作或命令**。例如按钮的点击，页面的滑动切换
- `V - View` **视图**：实际的 UI 组件
- `M - ViewModel + Model` **视图模型 + 数据层**：该层就是数据层，它一大一小有两层：
    - `ViewModel`: 例如需要展示在 `TextView` 上的文案、`ImageView` 上的图片资源等
    - `Repository / DataStore`: 例如需要通过数据库或网络请求得到的数据，它一般作为 `ViewModel` 的数据源

它们的关系如下所示：  
![](file:///Users/mantou/.config/joplin-desktop/resources/ac9766c4f4754c34a6ec43abdb2b83aa.png)  
对于 V 层和 M层 都是比较好理解的，而 I层 和我们之前所遇到的 `Controller`、 `Presenter` 有所不同，它不是一个单独具体的实体，而是一个描述数据流动的模型。

那么它要如何表示呢？

### [](#)[](#)2.4 Intent和响应式编程

MVI 认为只要视图还存在，用户就会源源不断地和视图界面进行交互，所以在 UI 的生命周期内会产生很多用户操作所产生出的数据。

这些源源不断数据则可以用**数据流**来表示，数据流模型可以简单的描述为**在生产者-消费者模型下，生产者作为上游可以不断产生数据，下游的消费者接收这些数据并进行处理消费**，而用户的交互和程序的响应正好能对应这个模型：

- **生产者 —— 数据流的起点 —— 用户的交互**，例如点击某个按钮， 它能产生一个信号，来让 App 去请求网络数据
- **消费者 —— 数据流的终点 —— UI**， 将网络请求回来的数据进行层层处理，然后显示在屏幕上

而 **Intent 就是生产者， 即数据流的起点**，例如下面代码就是用户由交互产生的一个意图：

```kotlin
button.setOnClickListener {
    // 产生一个意图
    viewModel.sendIntent(BUTTON_CLICK)
}

```

其次，处理数据流的模型是**响应式编程**模型， 我们知道一些专门的框架，例如 `RxJava`、`Flow`，这里不再具体介绍了…

所以这里的意图作为数据流的起点，也应该使用响应式编程的做法，所以我们一般看到的示例代码都是用 协程的 `Channel` 和 `StateFlow` 举例子的（这里不了解的同学可以看这篇文章：[深潜Kotlin协程（十六）：Channel](https://blog.csdn.net/rikkatheworld/article/details/125491766 "https://blog.csdn.net/rikkatheworld/article/details/125491766") 和 [深潜Kotlin协程（二十三 完结篇）：SharedFlow 和 StateFlow](https://blog.csdn.net/rikkatheworld/article/details/125665002 "https://blog.csdn.net/rikkatheworld/article/details/125665002")）：

```kotlin
button.setOnClickListener {
    lifecycleScope.launch {
       // 产生一个意图, ViewModel 使用一个 Channel 来接收意图
       viewModel.channel.send(BUTTON_CLICK)
    }
}

```

具体的代码可以放到第三节再看。

最后，MVI 框架已经初具雏形，它是一个 **单向数据流+唯一可信数据源+响应式编程**的模型，和 MVP、MVVM 相比，它主要的区别是引入了**数据流**这一概念，因为 Kotlin 的协程和 Jetpack 的支持，我们现在可以很舒服的在 Android 框架中使用响应式编程，所以这也是 MVI 为什么在 Android 框架上开始流行的原因。

它的数据流动可以用下面这张图来概括：  
![在这里插入图片描述](file:///Users/mantou/.config/joplin-desktop/resources/66d6f24626944243ac586c91f28ebfd8.png)

## [](#)[](#)3. 示例

我们可以基于 MVP 或者无 `DataBinding` 版本的 MVVM 搞出 MVI模式。 由于我们需要使用响应式编程，而 ViewModel 提供了协程作用域，方便于我们使用 `Flow`，所以 MVVM 相较于 MVP 能够更舒服的做出 MVI。**所以基本上所有的 MVI 架构都是用 MVVM 来写的，即使用 ViewModel 而非 Presenter。**

### [](#)[](#)3.1 定义和处理 Intent

意图描述用户交互，所以我们可以把所有用户有关的操作都写出来，并用 `场景名+Intent` 来命名，假设我们的界面是一个新闻列表页面：

```kotlin
// 新闻界面所有的用户操作, 基本类
sealed class NewsIntent

class UserClickNewsIntent(val url: String): NewsIntent() // 用户点击具体某个新闻Item的交互
object RefreshNewsIntent : NewsIntent()  // 用户刷新新闻列表的交互

```

并且在 VieawModel 中定义数据流的开端：

```kotlin
class NewsViewModel : ViewModel(){
    // 定义接收意图的通道
    val newsIntent = Channel<NewsIntent>()
    
    init {
        handleIntent()
    }

    private fun handleIntent() {
        viewModelScope.launch {
            newsIntent.consumeAsFlow().collect {
                when (it) {
                    is UserClickNewsIntent -> intoNewsItem(it.url)  // 处理 UserClickNewsIntent 意图
                    is RefreshNewsIntent -> fetchNews()  // 处理 RefreshNewsIntent 意图
                }
            }
        }
    }
    
    private suspend fun intoNewsItem(url: String) {
        ...
    }

    private suspend fun fetchNews() {
        ...
    }
}

```

### 3.2 触发 Intent

在 Activity / Fragment 这种第一层级的视图中，定义触发 `Intent` 的逻辑，一般是通过点击事件等操作，和 MVVM 中的触发逻辑一样，不过这里要在协程中触发，并且使用 `Channel` 或其它 Flow 工具，因为这样做是一种响应式编程的逻辑。

```kotlin
class NewsActivity : ComponentActivity() {
    private val viewModel: NewsViewModel = ViewModelProvider(this).get(NewsViewModel::class.java)

    override fun onCreate(savedInstanceState: Bundle?, persistentState: PersistableBundle?) {
        super.onCreate(savedInstanceState, persistentState)
        initListener()
    }

    private fun initListener() {
        refreshButton.setOnClickListener {
            sendIntent(RefreshNewsIntent)
        }
        createNewsItem(onItemClick = { newsItem ->
            sendIntent(UserClickNewsIntent(newsItem.url))
        })
    }
    
    private fun sendIntent(intent: NewsIntent) {
        lifecycleScope.launch {
            viewModel.newsIntent.send(intent)
        }
    }
}

```

### 3.3 定义 UiState 作为 View 的唯一数据源

我们通过归纳新闻页的页面状态，可以分成 初始态、加载中、加载成功、加载失败 四个状态，那么我们将状态收归到 UiState 中去，使用 `功能名+UiState` 来命名：

```kotlin
sealed class NewsUiState // 新闻页面的 UI 状态

object NewsUiStateInitial: NewsUiState() // 初始状态
object Loading: NewsUiState() // 正在加载
class LoadingSuccess(val newsList: List<NewsItem>): NewsUiState()  // 加载成功
class LoadingFail(val errorMessage: String): NewsUiState()  // 加载失败

```

ViewModel 持有 `NewsUiState`，并暴露出去，类似于 `LiveData` 那样子 ：

```kotlin
class NewsViewModel : ViewModel() {
    private val _newsUiState = MutableStateFlow<NewsUiState>(NewsUiStateInitial)
    val newsUiState: StateFlow<NewsUiState> = _newsUiState
}

```

Activity 依赖 ViewModel 持有的 UiState， 用于进行视图刷新：

```kotlin
class NewsActivity : ComponentActivity() {
    ...
    private fun observerUiState() {
        lifecycleScope.launch {
            // 这里使用 repeatOnLifecycle 包装一下性能更好
            viewModel.newsUiState.collect {
                when(it) {
                    is NewsUiStateInitial -> initial()
                    is Loading -> loading()
                    is LoadingSuccess -> updateNewsList(it.newsList)
                    is LoadingFail -> showError(it.errorMessage)
                }
            }
        }
    }

```

### 3.4 刷新 UiState

ViewModel 在处理完成后，通过更新 `newsUiState`，来触发 View 的刷新：

```kotlin
class NewsViewModel(private val dataStore: NewsDataStore = NewsDataStore()) : ViewModel() {
    private suspend fun fetchNews() {
        dataStore.fetchNews.flowOn(Dispatchers.Default)
            .catch {
                _newsUiState.value = LoadingFail("加载失败啦")
            }
            .collect {
                if (it.isEmpty()) _newsUiState.value = LoadingSuccess(it)
                else _newsUiState.value = LoadingFail("数据是空的")
            }
    }
    ..
}

data class NewsData(private val title: String)
class NewsDataStore() {
    // 用于获取新闻列表的 Flow
    val fetchNews: Flow<List<NewsData>> = flow {
        val news = fetchData()
        emit(news)
    }

    suspend fun fetchData(): List<NewsData> = api.getNewsData()
}

```

## 4. 总结

无论是 MVC、MVP、MVVM 还是 MVI，它们的共同点都是有 M层 和 V层。

所以这些架构的区别，也就是 MV 后面那个字母的区别，但万变不离其中的是：**它们的作用是描述 Model 和 View 之间的关系**。

- MVI 是基于 MVC 的发展，它是核心是**数据的单向流动**，优点是易追踪问题
- MVI 引入了数据流模型，所以使用了 **响应式编程**模型来处理数据流，而 `Intent` 意图，就是这个数据流的起点，即生产者，而 `View` 则是数据流的重点，即消费者
- MVI 中，整个 View 只依赖一个 `State` 刷新，这个 `State` 就是**唯一可信数据源**， 仅依赖单一状态的 UI 是更好测试的
- 在 Kotlin Anroid 中，因为 MVVM 很好的支持协程 `Flow`（响应式模型），所以我们一般用 MVVM 来写 MVI

综上所述，MVI 和 MVP、MVVM 这两位并不是一个维度的东西。MVI 强调**数据的流动方向**，而后两者则是强调结构分层。

MVI 的缺点是：

- UiState 的代码量容易随着视图的复杂度增加而增加
- UiState 爆炸时，UI 的更新点可能就会变多，频繁触发更新会导致内存消耗多， 这里需要实现一个状态的局部刷新，把更新频率尽可能降低

但是这些缺点， 大前端早就已经克服了，Android 肯定也有对应的实现，这里就不再赘述，后面遇到了再记录。

## [](#)[](#)参考

[MVI(Model-View-Intent) Pattern in Android](https://medium.com/code-yoga/mvi-model-view-intent-pattern-in-android-98c143d1ee7c "https://medium.com/code-yoga/mvi-model-view-intent-pattern-in-android-98c143d1ee7c")  
[官网](https://developer.android.com/jetpack/guide/ui-layer "https://developer.android.com/jetpack/guide/ui-layer")

[重学架构-MVI](https://juejin.cn/post/7050412734311038989#heading-22)

[用Kotlin Flow解决Android开发中的痛点问题](https://juejin.cn/post/7031726493906829319)

# [原文链接](https://blog.csdn.net/rikkatheworld/article/details/126472940)



