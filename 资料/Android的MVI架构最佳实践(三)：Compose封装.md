---
author: zjmantou
title: Android的MVI架构最佳实践(三)：Compose封装
time: 2024-02-29 周四
tags:
  - Android
  - 资料
  - MVI
---
## 前言

声明式UI的最佳搭档肯定是MVI了，例如前端的Flux或Redux。Compose作为Android官方的声明式UI框架已经非常成熟了，虽然目前从性能上无法碾压旧版本但是也在不断提升了，未来可期。此篇中我们就实现一个简单的脚手架消除样板代码，满足我们快速的构建一个MVI的Composable。

## Compose的State

首先我们处理UIState，在Compose中为了在重组中订阅保持的数据，会使用`State<out T>`，我们在composable中使用函数`val result = remember { }`来保持数据，防止重组中的性能消耗，但是ViewModel的特性就是生命周期内保持数据，因此我们只需要定义`androidx.compose.runtime.State`，对于纯Compose的项目我们修改BaseViewModel是再好不过：

```kotlin
abstract class BaseViewModel<S : State> : ViewModel() {
​
    abstract fun initialState(): S
    
    private val _viewState: MutableState<S> by lazy { mutableStateOf(initialState()) }
​
    val viewState: androidx.compose.runtime.State<S> = _viewState
}
```

## Composable中使用示例

```kotlin
sealed class TestState : State {
    object Loading : TestState()
    object Content : TestState()
}
​
class SampleViewModel : BaseViewModel<TestState>() {
    override fun initialState(): TestState  = TestState.Loading
}
​
@Composable
fun StateEffectScaffold() {
    val viewModel = viewModel<SampleViewModel>()
    when (viewModel.viewState) {
        TestState.Loading -> CircularProgressIndicator()
        TestState.Content -> Text("Content")
    }
}
```

## 兼容Fragment

大多时候我们还不是纯Compose开发，我们还需要一个兼容的`BaseViewModel`该如何做呢？官方也提供了很多扩展函数来实现这些需求：

- `LiveData` 转换 `Compose State`
```kotlin
@Composable
fun <T> LiveData<T>.observeAsState(): State<T?> = ...
```

- `flow` 转换 `Compose State`
```kotlin
@Composable
fun <T> Flow<T>.collectAsStateWithLifecycle(
    initialValue: T,
    lifecycleOwner: LifecycleOwner = LocalLifecycleOwner.current,
    minActiveState: Lifecycle.State = Lifecycle.State.STARTED,
    context: CoroutineContext = EmptyCoroutineContext
): State<T> = ...
```

这样我们之前篇幅中封装的`BaseViewModel`就可以通过这些api兼容到composable中了

### 通过ViewModel获得ComposeState

1. 若`BaseViewModel`中的state使用的是`StateFlow`。StateFlow要求必须有默认值，因此Compose的`uiState`始终有值。
```kotlin
// viewModel.state is StateFlow
val uiState = viewModel.state.collectAsStateWithLifecycle()
```

2. 若`BaseViewModel`中的state使用的是`SharedFlow(replay = 1)`。`LiveData和SharedFlow`一样没有默认值，因此Compose的`uiState`可空，官方提供的函数就要求初始化一个默认值，否则`state`就可能为**null**。
```kotlin
val initialState: S = ..
// viewModel.state is LiveDate
val uiState = viewModel.state.observeAsState(initial = initialState)
// viewModel.state is SharedFlow
val uiState = viewModel.state.collectAsStateWithLifecycle(
    initialValue = initialState
)
```

3. 优化`option2`中必传默认值参数问题。`LiveData和SharedFlow`不传递默认值会让`state`可空，我们过滤**null**不处理，那么默认值就变成可选参数了。

```kotlin
val initialState: S? = null  
val BaseViewModel<*,*,*>.replayState = state.replayCache.firstOrNull()  
val uiState = viewModel.state.collectAsStateWithLifecycle(  
   initialValue = replayState  
)  
(uiState.value ?: initialState)?.let { ... }
```

### State样板代码封装

Compose关注回调的state，而`@Composable`函数可以嵌套，那么我们可以简单封装获得一个脚手架让MVI使用更简单。甚至还可以套娃并共享同一个ViewModel来共享获取数据。

```kotlin
@Composable
fun <S : State, VM : BaseViewModel<*, S, *>> StateEffectScaffold(
    viewModel: VM,
    initialState: S? = null,
    lifecycle: Lifecycle = LocalLifecycleOwner.current.lifecycle,
    minActiveState: Lifecycle.State = Lifecycle.State.STARTED,
    context: CoroutineContext = EmptyCoroutineContext,
    content: @Composable (VM, S) -> Unit
) {
    val uiState = viewModel.state.collectAsStateWithLifecycle(
        initialValue = viewModel.replayState,
        lifecycle = lifecycle,
        minActiveState = minActiveState,
        context = context
    )
    (uiState.value ?: initialState)?.let { content(viewModel, it) }
}
​
//实战效果:
StateEffectScaffold(
    viewModel = hiltViewModel<ABaseViewModel>()
) { viewModel, state ->
    when (state) {
        TestState.Loading -> CircularProgressIndicator()
        TestState.Content -> Text("Content")
    }
}
```

## Compose的Effect

MVI中UIState分为`state`和`effect`，effect不需要state一样在重组中保持数据，在Compose中effect有专门的处理方式。再配合`repeatOnLifecycle`即可实现和Fragment中一样的效果。有时候我们不需要effect那么把这个函数参数作为可空传递。

```kotlin
@Composable
fun <E : Effect, VM : BaseViewModel<*, *, E>> StateEffectScaffold(
    viewModel: VM,
    lifecycle: Lifecycle = LocalLifecycleOwner.current.lifecycle,
    sideEffect: (suspend (VM, E) -> Unit)? = null,
) {
    sideEffect?.let {
        val lambdaEffect by rememberUpdatedState(sideEffect)
        LaunchedEffect(viewModel.effect, lifecycle) {
            lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.effect.collect { lambdaEffect(viewModel, it) }
            }
        }
    }
}
```

## Composable脚手架

上面我们分别处理了`State和Effect`，只需要稍加组合并且把一些参数全部暴露出来，就得到一个快速开发的脚手架了。这里用state是`SharedFlow`来演示代码

```kotlin
@Composable
fun <S : State, E : Effect, V : BaseViewModel<*, S, E>> StateEffectScaffold(
    viewModel: V,
    initialState: S? = null,
    lifecycle: Lifecycle = LocalLifecycleOwner.current.lifecycle,
    minActiveState: Lifecycle.State = Lifecycle.State.STARTED,
    context: CoroutineContext = EmptyCoroutineContext,
    sideEffect: (suspend (V, E) -> Unit)? = null,
    content: @Composable (V, S) -> Unit
) {
    sideEffect?.let {
        val lambdaEffect by rememberUpdatedState(sideEffect)
        LaunchedEffect(viewModel.effect, lifecycle, minActiveState) {
            lifecycle.repeatOnLifecycle(minActiveState) {
                if (context == EmptyCoroutineContext) {
                    viewModel.effect.collect { lambdaEffect(viewModel, it) }
                } else withContext(context) {
                    viewModel.effect.collect { lambdaEffect(viewModel, it) }
                }
            }
        }
    }
    // collectAsStateWithLifecycle 在横竖屏变化时会先回调initialState 所以必须把replay latest state传递过去
    val uiState = viewModel.state.collectAsStateWithLifecycle(
        initialValue = viewModel.replayState,
        lifecycle = lifecycle,
        minActiveState = minActiveState,
        context = context
    )
    (uiState.value ?: initialState)?.let { content(viewModel, it) }
}
```

## 脚手架使用演示

```kotlin
@Composable
fun TemplateScreen() {
    StateEffectScaffold(
        viewModel = hiltViewModel<TemplateViewModel>(),
        sideEffect = { viewModel, sideEffect ->
            when (sideEffect) {
                is TemplateEffect.ShowToast -> {
                    TODO("ShowToast ${sideEffect.content}")
                }
            }
        }
    ) { viewModel, state ->
        when (state) {
            TemplateState.Loading -> Loading()
            TemplateState.Empty -> Empty()
        }
    }
}
```

## AndroidStudio Live Template提升开发效率

IDEA 可以通过 `NEW -> Activity/Fragment`来选择一个模版快速生成一些代码，但是新版的AndroidStudio如果要自定义模版需要自己开发一个IDEA Plugin才可以做到。怎么既简单又能快速满足这个要求呢，那`Live Template`就可以发挥一定作用了，但是生成的代码在一个文件中需要自己手动分包。复制下面提供的模版到剪贴板，按图顺序操作。

	你输入`mvi`后立马自动获得下面模版中的代码，并且会等待你进一步输入操作。
	`$NAME$`是要输入主命名的占位符，你输入Home按下回车后，所有占位符位置会自动以Home替换，
	演示实战请看下节。

```kotlin
import androidx.compose.runtime.Composable
import androidx.hilt.navigation.compose.hiltViewModel
import com.arch.mvi.intent.Action
import com.arch.mvi.model.Effect
import com.arch.mvi.model.State
import com.arch.mvi.view.StateEffectScaffold
import com.arch.mvi.viewmodel.BaseViewModel
import dagger.hilt.android.lifecycle.HiltViewModel
import javax.inject.Inject

/**
 * - new build package and dirs
 *   - contract
 *   - viewmodel
 *   - view
 */

/** - contract */
sealed class $NAME$Action : Action {
    data object LoadData : $NAME$Action()
    data class OnButtonClicked(val id: Int) : $NAME$Action()
}

sealed class $NAME$State : State {
    data object Loading : $NAME$State()
    data object Empty : $NAME$State()
}

sealed class $NAME$Effect : Effect {
    data class ShowToast(val content: String) : $NAME$Effect()
}

/** - viewmodel */
@HiltViewModel
class $NAME$ViewModel @Inject constructor(
//    private val reducer: $NAME$Reducer,
//    private val repository: $NAME$Repository,
//    private val dispatcherProvider: CoroutineDispatcherProvider
) : BaseViewModel<$NAME$Action, $NAME$State, $NAME$Effect>() {

    init {
        sendAction($NAME$Action.LoadData)
    }

    override fun onAction(action: $NAME$Action, currentState: $NAME$State?) {
        when (action) {
            $NAME$Action.LoadData -> {
                /*viewModelScope.launch {
                    withContext(dispatcherProvider.io()) {
                        runCatching { repository.fetchRemoteOrLocalData() }
                    }.onSuccess {
                        emitState(reducer.reduceRemoteOrLocalData())
                    }.onFailure {
                        emitState($NAME$State.Empty)
                    }
                }*/$END$
            }

            is $NAME$Action.OnButtonClicked -> {
                emitEffect { $NAME$Effect.ShowToast("Clicked ${action.id}") }
            }
        }
    }
}

/** - view */
@Composable
fun $NAME$Screen() {
    StateEffectScaffold(
        viewModel = hiltViewModel<$NAME$ViewModel>(),
        sideEffect = { viewModel, sideEffect ->
            when (sideEffect) {
                is $NAME$Effect.ShowToast -> {
                    TODO("ShowToast ${sideEffect.content}")
                }
            }
        }
    ) { viewModel, state ->
        when (state) {
            $NAME$State.Loading -> {
                TODO("Loading")
            }
            $NAME$State.Empty -> {
                TODO("Empty")
            }
        }
    }
}
```

### 用Live Template快速开发

需求和最佳实践第一篇中一致: 登录页面点击登录按钮，请求网络返回登录结果，登录成功跳转，登录失败展示错误页面。因此MVI中ViewModel和Model的定义完全一样，唯一不同就是View使用Composable。

1. 在AS新建一个空的kotlin文件，输入`mvi`获得模版，输入Logon
2. 修改`Action`的模版代码为需求中的action，修改`State和Effect`和需求中UIState匹配
```kotlin
sealed class LogonAction : Action {
    data object OnButtonClicked : LogonAction()
}

sealed class LogonState : State {
    data object LogonHub : LogonState()
    data object Loading : LogonState()
    data object Error : LogonState()
}

sealed class LogonEffect : Effect {
    data object Navigate : LogonEffect()
}
```

3. 修改ViewModel代码
```kotlin
class LogonViewModel : BaseViewModel<LogonAction, LogonState, LogonEffect>() {
        override fun onAction(action: LogonAction, currentState: LogonState?) {
            when (action) {
                LogonAction.OnButtonClicked -> {
                    flow {
                        kotlinx.coroutines.delay(2000)
                        emit(Unit)
                    }.onStart { 
                        emitState(LogonState.Loading)
                    }.onEach {
                        emitEffect(LogonEffect.Navigate)
                    }.catch {
                        emitState(LogonState.Error)
                    }.launchIn(viewModelScope)
                }
            }
        }
    }
```

4. 构建的不同State下的Composable，实现sideEffect处理导航事件
```kotlin
@Preview
@Composable
fun LogonScreen() {
    val scaffoldState = rememberScaffoldState()
    StateEffectScaffold(viewModel = hiltViewModel<LogonViewModel>(),
        initialState = LogonState.LogonHub,
        sideEffect = { viewModel, sideEffect ->
            when (sideEffect) {
                LogonEffect.Navigate -> {
                    scaffoldState.snackbarHostState.showSnackbar("navigate")
                }
            }
        }
    ) { viewModel, state ->
        Scaffold(
            scaffoldState = scaffoldState
        ) {
            Box(modifier = Modifier.fillMaxSize().padding(it), contentAlignment = Alignment.Center) {
                when (state) {
                    LogonState.Loading -> CircularProgressIndicator()
                    LogonState.Error -> Text(text = "error")
                    LogonState.LogonHub -> Button(onClick = {
                        viewModel.sendAction(LogonAction.OnButtonClicked)
                    }) {
                        Text(text = "logon")
                    }
                }
            }
        }
    }
}
```

## 总结

此篇中我们对Compose中使用MVI架构模式做了详细讲解，并且考虑非纯Compose的项目的兼容。最后对Composable简单的封装一个脚手架用于快速构建Compose的MVI结构，考虑到每次书写的模版代码太多又使用了`Live Template`来再次提升构建速度。最终用实战项目来检验他们。由于大部分开发者并不会再实际开发环节书写UT，导致大家对MVI的优点易于测试感知不强，下一章开始主要讲代码质量管理环节中的MVI的`Unit test(单元测试)`。

# 原文链接
[Android的MVI架构最佳实践(三)：Compose封装](https://hoooopy.com/index.php/2023/10/25/android%e7%9a%84mvi%e6%9e%b6%e6%9e%84%e6%9c%80%e4%bd%b3%e5%ae%9e%e8%b7%b5%e4%b8%89%ef%bc%9acompose%e5%b0%81%e8%a3%85/)

