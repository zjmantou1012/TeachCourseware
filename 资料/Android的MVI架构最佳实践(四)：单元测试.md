---
author: zjmantou
title: Android的MVI架构最佳实践(四)：单元测试
time: 2024-02-29 周四
tags:
  - 资料
  - Android
  - MVI
---
# Android的MVI架构最佳实践(四)：单元测试

## 前言

Bug是我们任何时候绕不开的问题，因此我们的团队中通常都会有测试，开发过程中有自测、交付测试等流程。单元测试就是一种保障交付质量的利器，先看看GPT怎么说的，单元测试是一种测试方法，它用于测试程序中的最小可测试单元，例如函数、方法或类的行为。它旨在验证单元的行为是否符合预期，并帮助开发人员在早期发现和修复缺陷。单元测试通常由开发人员编写，并使用测试框架进行自动化执行。测试框架可以提供断言和模拟工具来简化测试编写过程。在编写单元测试时，开发人员应该尽可能考虑各种情况和输入，以确保代码的质量和可靠性。

## 概念入门

单元测试你可以把它理解为对一个函数方法中的逻辑进行拆解测试，很多时候单元测试的用例设计是和你设计这个函数功能时候的思路是一致的。例如有个功能是对传入数字进行累加，那么设计时候我们就考虑传入的数字是负数、正数和0，那么设计的验证测试用例也要有3个。 单元测试中一般我们会有3个大步骤：

1. 提供上下文：准备测试数据和测试环境，以便在测试中使用。这包括创建对象、设置属性、调用方法等。
    
2. 执行测试代码：执行要测试的代码，通常是调用某个方法或函数。
    
3. 断言验证测试结果

## Android中的测试单元

其实在android的一个project中单元测试可以分为仪器化测试(UI测试)和逻辑单元测试，通常称呼的单元测试单指逻辑单元测试。一般我们会把这部分代码写在`app/src/test`下面，这部分代码运行时在本地机器JVM上并不是手机或者模拟器上，因此会缺失android framework支持因此无法测试网络，蓝牙，Wi-Fi等硬件支持的功能。因此一般我们需要对View层进行抽象只测试UI方法是否被调用，以此来确定逻辑正确性完成单元测试。像MVC几乎无法完成单元测试，在java中一般我们对整个类的每行代码进行单元测试覆盖，而kotlin中我们一般以函数作为最小维度覆盖。

1. 添加单元测试依赖。在build.gradle的依赖中
    
    testImplementation
    
    可以单独给单元测试提供依赖库，通常单元测试都是借助Junit来帮忙

```gradle
//Junit5支持
testImplementation "org.junit.jupiter:junit-jupiter-api:5.8.2"
testRuntimeOnly "org.junit.jupiter:junit-jupiter-engine:5.8.2"
testImplementation "org.junit.jupiter:junit-jupiter-params:5.8.2"
// ArchTaskExecutor
testImplementation "androidx.arch.core:core-testing:2.2.0"
//协程支持
testImplementation "org.jetbrains.kotlinx:kotlinx-coroutines-test:1.7.3"
```


2. 快速创建单元测试。使用下面示例代码进行单元测试。可以通过如下操作快速创建，选中要测试的方法 右键 -> Generater -> Test -> 选择单元测试为Junit5

```kotlin
private fun logon() {
    repo.getRandomNumber()
        .onStart {
            emitState(LogOnState.Loading)
        }.catch { ex ->
            emitState(LogOnState.Error(ex))
        }.onEach { result ->
            if (result == 0) {
                emitState(LogOnState.Empty)
            } else {
                emitState(LogOnState.Data(result))
            }
        }.launchIn(viewModelScope)
}

```

3. 测试用例，尽可能的对方法参数和返回值的情况进行覆盖。这里举例2个case。

	注意单元测试的方法名推荐用 `Give xxx When xxx And xx Then xx`这样的语句来作为命名，这样可以直观的知道这个单元测试的用例详情。Given 给出上下文（准备数据）；When 执行测试代码；Then 断言测试结果。

```kotlin
internal class LogonTest {
​
    //case: 测试当随机数为0时，是否返回Empty状态。
    @Test
    fun `test when getRandomNumber is zero then LogOnState_Empty`() = runTest {
        // 设置Mock Repository对象的行为
        val repo = object : Repository(){
        override fun getRandomNumber() = flowOf(0)
        }
        // 创建ViewModel对象
        val viewModel = MyViewModel(repo)
​
        // 调用logon()函数
        viewModel.logon()
​
        // 验证ViewModel的状态是否符合预期（first是一个挂起函数需要协程test库 runTest{}）
        val state = viewModel.state.first()
        assertEquals(LogOnState.Empty, state)
    }
​
    //case: 测试当随机数不为0时，是否返回Data状态。
    @Test
    fun `test when getRandomNumber has value thenReturn LogOnState_Data`() {
        val repo = object : Repository(){
        override fun getRandomNumber() = flowOf(40)
        }
        val viewModel = MyViewModel(repo)
        // 调用logon()函数
        viewModel.logon()
​
        // 验证ViewModel的状态是否符合预期
        val state = viewModel.state.first()
        assertEquals(LogOnState.Data(40), state)
    }
}
```

## Mockito-kotlin

[Mockito](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fmockito%2Fmockito-kotlin)是一个用于Java的开源Mock框架。它允许您使用简单的API来创建Mock对象，并使用这些对象来模拟真实对象的行为。Mockito可以用于单元测试和集成测试，它可以与JUnit和TestNG等测试框架一起使用。主要作用就是方便在测试中模拟一些在本单元测试无需覆盖或者framework api无法支持的情况。

首先添加依赖

```kotlin
testImplementation "org.mockito.kotlin:mockito-kotlin:5.1.0"
//junit5 support
testImplementation "org.mockito:mockito-junit-jupiter:4.6.1"
```

用ViewModel的参数`LogonRepo`来学习用法

```kotlin
//数据接口
interface LogonRepo {
    val testInt: Int
    fun getRandomNumber(): Flow<Int>
    suspend fun getTestInt(): Int
}
​
//ViewModel
class LogonViewModel constructor(
    private val repo: LogonRepo
){
    fun logon() = repo.get()...
}
​
//构造ViewModel使用数据接口实现类
val viewModel = LogonViewModel(repo = MyLogonRepo())
```

由于单元测试需要函数或者类功能单一才会易于测试，这也比较符合程序设计中的单一职责原则。因此很多时候我们会对`LogonRepo`抽象成接口，然后对实现类独编写单元测试。那么我们在ViewModel的单元测试中就不需要再对实现类进行单元测试了，那么我们可以借助`Mockito`来模拟一些实现类的属性或者方法的返回值。

```kotlin
//获得一个mock对象
val repo = mock<LogonRepo>()
​
//mock 属性
whenever(mock.testInt).thenReturn(9)
println(mock.testInt) // 9
​
//mock 方法
whenever(mock.getRandomNumber()).thenReturn(flowOf(6))
​
//suspend方法mock
wheneverBlocking { repository.getTestInt() }.doReturn(10)
//或者下面这样
repository.stub { 
    onBlocking { getTestInt() }.doReturn(10)
}
```

如果我们的逻辑中出现了例如`SharePreference，Context.getString`这些framework提供的api，这时候单元测试会因为运行时异常无法执行，这时候也可以借助`Mockito`来模拟。

```kotlin
val mock = mock<Context>()
whenever(mock.getString(R.string.app_name)).thenReturn("app-name")
​
val mock = mock<Context>()
val mockSP = mock<SharedPreferences>()
whenever(mock.getSharedPreferences("pre", Context.MODE_PRIVATE)).thenReturn(mockSP)
whenever(mockSP.getString("app_key", "")).thenReturn("key01")
```

当逻辑中执行SP的`getString("app_key", "")`时候就拿到的是mock值`key01`，并不是真实的sp储存值。whenever是Mockito框架中的一个方法，它用于设置Mock对象的行为。在这个例子中，我们使用whenever方法来设置Mock对象在调用getString方法时的行为。any()是Mockito框架中的一个方法，它表示任意类型的参数。因此，这个代码的意思是，当调用Mock对象的getString方法时，无论传入什么参数，都返回字符串"key01"。在下面的代码中，我们可以看到当传入参数为"a"和"b"时，getString方法都返回字符串"key01"。


```kotlin
whenever(mockSP.getString(any(), any())).thenReturn("key01")

println(mockSP.getString("a", "")) //"key01"
println(mockSP.getString("b", "")) //"key01"
```

## 单元测试中的多线程问题

在单元测试中，多线程可能会导致测试失败或产生不可预测的结果。因此在单元测试中我们要想办法构建出一个单线程的环境，让多线程的Task的`Runnable.run()`在当前线程直接执行。让我们看看kotlin的协程和Jetpack的ArchTaskExecutor如何处理。

1. kotlin中线程切换一般我们会使用协程，因此官方也提供了测试辅助库。首先是`runBlockingTest`可以让suspend函数在当前线程阻塞式执行，其次`Dispatchers.setMain`是Kotlin协程库中的一个函数，它用于将当前线程设置为主线程，以便在单元测试中测试协程代码。当我们在单元测试中使用协程时，我们需要在测试代码中模拟主线程的行为，从而避免在测试中出现线程问题。（在最新的1.9.0版本中`runBlockingTest`被修改为`runTest`，参见：[github.com/Kotlin/kotl…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FKotlin%2Fkotlinx.coroutines%2Fblob%2Fmaster%2Fkotlinx-coroutines-test%2FMIGRATION.md%EF%BC%89)

```kotlin
//依赖
testImplementation "org.jetbrains.kotlinx:kotlinx-coroutines-test"
//示例
@Test
fun testCoroutineCode() = runBlockingTest {
    Dispatchers.setMain(TestCoroutineDispatcher())
    val result = async { myFunction() }.await()
    assertEquals("expected result", result)
    Dispatchers.resetMain()
}

suspend fun myFunction(): String {
    // some async code
    return "expected result"
}
```

2. Androidx Jetpack库中的一个方法`ArchTaskExecutor.getInstance().setDelegate`，它用于在单元测试中模拟主线程和后台线程的行为。这个方法的作用是设置一个代理，用于替换默认的`TaskExecutor`，从而可以控制任务在哪个线程上执行。例如，以下代码演示了如何在单元测试中使用：

```kotlin
@Test
fun testTaskExecutor() {
    ArchTaskExecutor.getInstance().setDelegate(object : TaskExecutor() {
        override fun executeOnDiskIO(runnable: Runnable) {
            runnable.run()
        }

        override fun postToMainThread(runnable: Runnable) {
            runnable.run()
        }

        override fun isMainThread(): Boolean {
            return true
        }
    })

    // test code
    //LiveData().postValue(xxxxx)
    ArchTaskExecutor.getInstance().setDelegate(null)
}
```

在这个示例中，我们使用`ArchTaskExecutor.getInstance().setDelegate`方法设置一个代理，用于模拟主线程和后台线程的行为。在代理中，我们重写了`executeOnDiskIO`、`postToMainThread`和`isMainThread`这三个方法，以便控制任务在当前线程上执行。在`executeOnDiskIO`和`postToMainThread`方法中，我们直接调用了`run`方法来执行任务，从而避免在测试中出现线程问题。在`isMainThread`方法中，我们返回`true`，表示当前线程是主线程。最后，我们在测试代码执行完毕后，使用`ArchTaskExecutor.getInstance().setDelegate(null)`方法将代理重置为默认值。这样，我们就可以在单元测试中模拟主线程和后台线程的行为，而不用担心线程问题。

## Junit5的BeforeEach、AfterEach和ExtendWith

使用上面的知识我们可以解决90%单元测试中因为线程问题导致的错误，但是每个单元测试方法中都要写这些代码太麻烦了。Junit也提供了api来消除这些重复代码

- `BeforeEach`和`AfterEach`是Junit 5框架中的注解，它们用于在每个测试方法执行前和执行后执行一些通用的逻辑。这些注解可以用于执行一些初始化或清理操作，例如初始化测试数据、关闭数据库连接等。以下是一个示例：
```kotlin
@ExtendWith(MockitoExtension::class)
class LogonViewModelTest {

    lateinit var viewModel: LogonViewModel

    @Mock
    lateinit var repository: LogonRepo

    @BeforeEach
    fun setUp() {
        //每执行一个单元测试先执行此处代码
        viewModel = LogonViewModel(repository)
    }

    @AfterEach
    fun tearDown() {
        //每完成一个单元测试执行此处代码
    }   

    @Test
    fun `test logon`() = runTest {
        whenever { repository.getRandomNumber() } doReturn { flowOf(40) }
        viewModel.sendAction(LogOnAction.OnLogOnClicked)
        val state = viewModel.state.first()
        assertEquals(LogOnState.Data(40), state)
    }
}
```

- `@ExtendWith`在上面的示例中，我们使用`@ExtendWith(MockitoExtension::class)`注解来扩展测试类的行为，使用Mockito进行模拟。在测试类中，我们使用@Mock注解来创建一个LogonRepo对象的模拟实例。在test方法中，我们可以使用repository进行测试。这样，我们就可以在测试中使用模拟对象，而不用担心外部依赖的影响。除了Mockito，还有许多其他的扩展可以使用`@ExtendWith`注解进行引入。

```kotlin
// junit5 CoroutineTestExtension
@OptIn(ExperimentalCoroutinesApi::class)
class CoroutineTestExtension : BeforeEachCallback, AfterEachCallback {

    override fun beforeEach(context: ExtensionContext?) {
        Dispatchers.setMain(testDispatcher)
    }

    override fun afterEach(context: ExtensionContext?) {
        Dispatchers.resetMain()
    }

    companion object {
        val testDispatcher: TestDispatcher = UnconfinedTestDispatcher()
    }
}
// junit5 InstantTaskExecutorExtension
class InstantTaskExecutorExtension : BeforeEachCallback, AfterEachCallback {
    override fun beforeEach(context: ExtensionContext) {
        ArchTaskExecutor.getInstance().setDelegate(object : TaskExecutor() {
            override fun executeOnDiskIO(runnable: Runnable) = runnable.run()
            override fun postToMainThread(runnable: Runnable) = runnable.run()
            override fun isMainThread(): Boolean = true
        })
    }
    override fun afterEach(context: ExtensionContext?) {
        ArchTaskExecutor.getInstance().setDelegate(null)
    }
}
//使用示例
@ExtendWith(
    value = [
        CoroutineTestExtension::class,
        InstantTaskExecutorExtension::class
    ]
)
class LogonViewModelTest {
    ...
}
```
## LiveData单元测试

1. LiveData测试需要用到的lifecycle，因此需要用到`observeForever`函数。
```kotlin
class ShareViewModel : ViewModel() {
    val testLiveData = MutableLiveData<Int>()
}

//Unit test
import org.junit.jupiter.api.Assertions
import org.junit.jupiter.api.Test

class ShareViewModelTest {
    @Test
    fun getShareContact() {
        //Give
        val viewModel = ShareViewModel()
        //When
        viewModel.testLiveData.value = 12
        val values = mutableListOf<Int>()
        viewModel.testLiveData.observeForever {
            values.add(it)
        }
        //Then
        Assertions.assertEquals(12, values.first())
    }
}
```

运行时候发现报错了`java.lang.RuntimeException: Method getMainLooper in android.os.Looper not mocked.`。因为主线程的判断用到了Android framework api，需要用到`ArchTaskExecutor.getInstance().setDelegate`，这里就要用到上面提到过的`InstantTaskExecutorExtension`。给TestClass添加`@ExtendWith(InstantTaskExecutorExtension::class)`即可。

2. 借助`Mockito的verify`，在这个测试中，我们使用Mockito框架来创建一个Observer对象，使用verify()方法来验证Observer对象的onChanged()方法是否被调用，并且传递的参数是否为我们期望的值。
```kotlin
@ExtendWith(
    value = [
        MockitoExtension::class,
        InstantTaskExecutorExtension::class
    ]
)
internal class ShareViewModelTest {
    @Mock
    lateinit var observer: Observer<Int>

    @Test
    fun testUserLiveData() {
        val viewModel = ShareViewModel()
        viewModel.testLiveData.postValue(12)
        viewModel.testLiveData.observeForever(observer)
        verify(observer).onChanged(12)
    }
}
```

## Flow单元测试

官方提供了[Flow测试指导](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.android.com%2Fkotlin%2Fflow%2Ftest%3Fhl%3Dzh-cn)，这里提供一个简易切好用的封装用来测试Flow。首先需要知道Flow最终通过`suspend fun collect(collector: FlowCollector<T>)` 来收集数据，是一个挂起方法因此需要用到上面的 `runTest`来提供一个`TestScope`

```kotlin
/**flow testing function*/
internal fun <T> Flow<T>.toListTest(testScope: TestScope): List<T> {
    return mutableListOf<T>().also { values ->
        testScope.launchTest {
            this@toListTest.toList(values)
        }
    }
}

internal fun TestScope.launchTest(block: suspend CoroutineScope.() -> Unit) =
    this.backgroundScope.launch(
        context = UnconfinedTestDispatcher(this.testScheduler),
        block = block
    )
```

测试一个数据流，以及测试ViewModel中的state

```kotlin
@Test
fun testFlow() = runTest {
    val flow = (1..10).asFlow()
    val testResult = flow.toListTest(this)
    Assertions.assertEquals(1, testResult.first())
    Assertions.assertEquals(6, testResult[5])
    Assertions.assertEquals(10, testResult.last())
}

@Test
fun `test when OnLogOnClicked then LogonStateData`() = runTest {
    val states = viewModel.state.toListTest(this)
    whenever(repository.getRandomNumber()).doReturn(flowOf(40))
    viewModel.sendAction(LogOnAction.OnLogOnClicked)
    assertEquals(LogOnState.Data(40), states.first())
}
```

## MVI的优势

从上面的简单例子其实并不能反应单元测试的难易，很多时候因为类在构造时候依赖的对象和函数内部的复杂度会影响我们的单元测试书写难度。为啥MVI会易测试呢，首先我们来分析下上面的单元测试。发现有一个特点，单元测试就是用一个输入来获取一个输出，然后对输出结果进行断言。这不正符合单线数据流的思想么，通过前面的文章和知识，MVI的特点就是单向数据流；纯函数式编程；可观察数据流；易于模拟数据，所以MVI在单元测试中非常简单。

```kotlin
class LogonTest {
    @Test
    fun `test mvi`() = runTest {
        val repo = object : Repository(){
           override fun getRandomNumber() = flowOf(40)
        }
        val viewModel = MyViewModel(repo)
        viewModel.sendAction(LogOnAction.OnLogOnClicked)
        val state = viewModel.state.first()
        assertEquals(LogOnState.Data(40), state)
    }
}
```

## 单元测试覆盖率

单元测试覆盖率是指测试用例覆盖代码的百分比，是单元测试中逻辑和case设计完成度检查的重要指标。常用的代码覆盖率工具包括JaCoCo和Emma。下面是AS自带的`Run Tests in xxx with Coverage`生成报告和代码中行覆盖提示，没有覆盖行和条件有红色指示，绿色表示已经覆盖。

一些提高测试覆盖率的建议：

1.编写足够的测试用例，以覆盖代码的各个分支和边界条件。

2.使用Mockito等框架来模拟依赖项，以便更容易地编写测试用例。

3.使用覆盖率工具来测量测试用例覆盖的代码百分比，并根据需要进行修改和扩展测试用例。

4.使用代码静态分析工具来查找未覆盖的代码，并编写测试用例来覆盖这些代码。

总之，测试覆盖率是一个重要的指标，可以帮助我们确定测试用例是否足够覆盖代码。通过编写足够的测试用例和使用覆盖率工具，我们可以提高测试覆盖率，并确保代码的质量和稳定性。在商业项目中JaCoCo和SonarQube一起使用，以提高代码质量和测试覆盖率。JaCoCo可以生成测试覆盖率报告，并将其与SonarQube集成，以便在SonarQube中查看测试覆盖率指标和报告。SonarQube还可以使用JaCoCo生成的测试覆盖率数据来计算代码覆盖率指标，并提供更全面的代码质量分析和报告。

## 代码质量管理Jacoco&SonarQube

- [JaCoCo](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jacoco.org%2Fjacoco%2F)是一个Java代码覆盖率库，它可以帮助开发人员了解他们的代码库中哪些代码已经被测试覆盖，哪些代码还需要进行测试。JaCoCo可以与各种构建工具（如Maven和Gradle）集成，以生成代码覆盖率报告。
- [SonarQube](https://link.juejin.cn/?target=https%3A%2F%2Fwww.sonarqube.org%2F)是一个开源的代码质量管理平台，它可以帮助开发人员在整个开发周期中管理和提高代码质量。SonarQube可以与各种编程语言（如Java、C#、JavaScript等）集成，并提供了一系列功能，包括代码质量分析、代码覆盖率、代码复杂度、代码重复性、安全漏洞等方面的检测。
- 我们要利用这2个工具平台，使用构建工具Gradle给Android项目打造一个可视化代码覆盖率，首先，使用JaCoCo生成代码覆盖率报告，然后将报告传递给SonarQube进行分析。SonarQube将使用这些报告来提供更广泛的代码质量分析，包括代码复杂度、代码重复性、安全漏洞等方面的检测。

## 总结

本文主要介绍了MVI架构在Android开发中的单元测试最佳实践。介绍了如何使用覆盖率工具来测量测试用例覆盖的代码百分比，并提供了一些提高测试覆盖率的建议。通过本文的学习，我们可以了解到如何使用MVI架构来编写易于测试的Android应用程序，并提高代码质量和可维护性。个人通过项目实践总结出单元测试优点:

- 提高代码质量：单元测试可以帮助开发人员及时发现代码中的问题，例如逻辑错误、边界条件错误等，从而提高代码的质量。
    
- 提高代码可维护性：单元测试可以帮助开发人员更好地理解代码的功能和实现，从而提高代码的可维护性。
    
- 提高代码重构的安全性：单元测试可以帮助开发人员在重构代码时及时发现问题，从而提高重构的安全性。
    
- 提高开发效率：单元测试可以帮助开发人员快速定位和解决问题，从而提高开发效率。
    
- 促进团队协作：单元测试可以帮助团队成员更好地理解代码的实现和功能，从而促进团队协作。
    
- 降低维护成本：单元测试可以帮助开发人员及时发现问题，从而降低维护成本。检查新加逻辑是否影响到老的逻辑或者功能，防止老代码被篡改出现Bug。

# 原文链接

[Android的MVI架构最佳实践(四)：单元测试](https://hoooopy.com/index.php/2023/10/25/android%e7%9a%84mvi%e6%9e%b6%e6%9e%84%e6%9c%80%e4%bd%b3%e5%ae%9e%e8%b7%b5%e5%9b%9b%ef%bc%9a%e5%8d%95%e5%85%83%e6%b5%8b%e8%af%95/)
