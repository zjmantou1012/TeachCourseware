---
author: zjmantou
title: Android的MVI架构最佳实践(五)：UI测试
time: 2024-02-29 周四
tags:
  - 资料
  - Android
  - MVI
---
## 前言

首先需要明确UI测试在什么项目中可以发挥最大价值，在官方的测试文档中定义了测试金字塔（如图）：

- 小型测试是指单元测试，用于验证应用的行为，一次验证一个类。
    
- 中型测试是指集成测试，用于验证模块内堆栈级别之间的互动或相关模块之间的互动。
    
- 大型测试是指端到端测试，用于验证跨越了应用的多个模块的用户操作流程。

沿着金字塔逐级向上，从小型测试到大型测试，各类测试的保真度逐级提高，但维护和调试工作所需的执行时间和工作量也逐级增加。因此单元测试基本可以满足大部分的项目质量管理，UI测试占比不需要很大也可以没有。实践中UI测试的编码和维护工作量会较大，对于UI界面不会经常修改和SDK性质的**大项目**会比较合适。如果你们的项目功能基本稳定（没吊事做），但是需要对项目质量进行提升，那么UI测试的引入可以给你们带来kpi。

## UI 测试中的网络数据控制

由于UI测试和单元测试一样，最终断言的都是输出结果和期望结果。一般这个期望结果是个死值，只是在UI测试中这个值是一个View的状态或者整个界面的快照等，如果我们使用真实的网络环境数据，会导致输出结果的不定性，所以这个时候一般我们会Mock数据，用来固定api的输入数据获取稳定的输出结果。如果项目使用了OKhttp，可以借助OKhttp的拦截器或者mock server去完成这个工作。

1. Mockwebserver 添加依赖
```groovy
androidTestImplementation "com.squareup.okhttp3:mockwebserver:4.11.0"
```
2. 指定Okhttp请求的BaseURL为`localhost:8000`
3. 启动mock server
```kotlin
private var mockWebServer: MockWebServer = MockWebServer()
​
 override fun beforeEachTest() {
     mockWebServer = MockWebServer()
     mockWebServer.start(8000)
 }
​
 override fun afterEachTest() {
     mockWebServer.shutdown()
 }
```

4. 拦截请求返回mock response
```kotlin
mockWebServer.dispatcher = object : Dispatcher() {
     override fun dispatch(request: RecordedRequest): MockResponse {
         ....
         MockResponse()
             .setResponseCode(200 or 400)
             .setBody(...)
     }
}
```
5. mock server像拦截器。我们为了扩展性可以把response做成json配置，修改json文件即可获得不同的测试结果。

## Espresso

通常称呼的UI测试单指仪器化测试，是运行在设备上的测试，因此需要设备支持模拟器或者真机等。在Android应用程序中，UI测试可以使用Android框架[Espresso](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.android.com%2Ftraining%2Ftesting%2Fespresso%3Fhl%3Dzh-cn)提供的UI测试框架进行测试。UI测试通常涉及模拟用户与应用程序交互，并检查应用程序的响应是否正确。作为单元测试的一种，同样需要3个大步骤：

1. 提供上下文：准备测试数据和测试环境，以便在测试中使用。启动Activity/Fragment，mock api Server，多语言环境等。
    
2. 执行测试代码：执行要测试的步骤，通常是模拟用户的真实操作：点击滑动等。
    
3. 断言验证测试结果：断言View的展示内容等
    

Espresso 框架也是围绕上面的3大要素设计API，主要组件包括：

- Espresso - 用于与视图交互（通过 onView() 和 onData()）的入口点。此外，还公开不一定与任何视图相关联的 API，如 pressBack()。
- ViewMatchers - 实现 Matcher`<? super View>` 接口的对象的集合。您可以将其中一个或多个对象传递给 onView() 方法，以在当前视图层次结构中找到某个视图。
- ViewActions - 可以传递给 ViewInteraction.perform() 方法的 ViewAction 对象的集合，例如 click()。
- ViewAssertions - 可以通过 ViewInteraction.check() 方法传递的 ViewAssertion 对象的集合。在大多数情况下，您将使用 matches 断言，它使用视图匹配器断言当前选定视图的状态。

例如检测点击View后是否显示，需要先找出这个期望的View，然后模拟点击，最后断言View的显示状态。

```kotlin
onView(withId(R.id.my_view))
    .perform(click())
    .check(matches(isDisplayed()))
```

## Google Compose的UI测试指导

在 Compose 中只有一些可组合项会向界面层次结构中发出界面，因此需要采用不同的方法来匹配界面元素。官方也给出了UI测试指导[developer.android.com/jetpack/com…](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.android.com%2Fjetpack%2Fcompose%2Ftesting%3Fhl%3Dzh-cn%E3%80%82) 添加测试库的依赖，api与元素交互的方式主要有以下三种：

- 查找器：可供您选择一个或多个元素（或语义树中的节点），以进行断言或对其执行操作。
    
- 断言：用于验证元素是否存在或者具有某些属性。
    
- 操作：会在元素上注入模拟的用户事件，例如点击或其他手势。
    

编码的难度主要在查找器上面，我们需要找到测试的节点，最简单最有效的方法就是给Composeable添加一个test标记 `semantics { testTag = xxx }`，或者使用它的扩展函数`Modifier.testTag`
```kotlin
Button(
    modifier = Modifier
        .fillMaxWidth()
        .semantics { testTag = "test" }
        //.testTag("test"),
    onClick = {...}
) {
    Text(text = "按钮")
}
```

UI测试示例

```kotlin
class MyComposeTest {
    @get:Rule
    val composeTestRule = createComposeRule()
    // use createAndroidComposeRule<YourActivity>() if you need access to an activity
​
    @Test
    fun myTest() {
        // Start the app
        composeTestRule.setContent {
            MyAppTheme {
                MainScreen()
            }
        }
        //查找Button 并且点击
        composeTestRule.onNode(hasTestTag("test")).performClick()
        composeTestRule.onNodeWithText("Welcome").assertIsDisplayed()
    }
}
```

## Kaspresso UI test 框架

[Kaspresso](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FKasperskyLab%2FKaspresso)，Kaspresso是一个基于Espresso的Kotlin DSL框架，用于编写Android UI自动化测试，并且已经支持了Compose。它提供了一些简单易用的API，可以帮助开发人员编写可读性更高、可维护性更好的测试用例。

Kaspresso的主要特点包括：

- Kotlin DSL：使用Kotlin语言编写的DSL，可以提高测试用例的可读性和可维护性。
    
- 自动化等待：Kaspresso可以自动等待UI元素的出现和消失，无需手动编写等待逻辑。
    
- 屏幕截图：Kaspresso可以自动截取屏幕截图，方便开发人员调试测试用例。
    
- 异常处理：Kaspresso可以自动处理Espresso中的一些常见异常，例如：NoMatchingViewException、AmbiguousViewMatcherException等
    

### 集成

```groovy
dependencies {
    androidTestImplementation 'com.kaspersky.android-components:kaspresso:<latest_version>'
    // Allure support
    androidTestImplementation "com.kaspersky.android-components:kaspresso-allure-support:<latest_version>"
    // Jetpack Compose support
    androidTestImplementation "com.kaspersky.android-components:kaspresso-compose-support:<latest_version>"
}
```

### Kaspresso和Espresso对比

Espresso:

```kotlin
@Test
fun testFirstFeature() {
    onView(withId(R.id.toFirstFeature))
        .check(ViewAssertions.matches(
               ViewMatchers.withEffectiveVisibility(
                       ViewMatchers.Visibility.VISIBLE)))
    onView(withId(R.id.toFirstFeature)).perform(click())
}
```

Kaspresso-xml:
```kotlin
@Test
fun testFirstFeature() {
    MainScreen {
        toFirstFeatureButton {
            isVisible()
            click()
        }
    }
}
```

Kaspresso-compose:

```kotlin
@Test
fun testFirstFeature() {
    ComposeScreen.onComposeScreen<MainScreen>(composeTestRule) {
        toFirstFeatureButton {
            performClick()
            assertIsDisplayed()
        }
    }   
}
```

## screenshot 测试

这个测试比较特殊，它会融合你的UI测试代码在你设定UItest中对屏幕截图，然后等UI测试完成后对所有截图和期望的截图进行对比，主要检测UI的像素是否一致。相对UI test的对View元素的单一断言测试，截图测试会更加安全可靠。支持的框架有如下：

- [facebook-screenshot-tests-for-android](https://link.juejin.cn/?target=https%3A%2F%2Ffacebook.github.io%2Fscreenshot-tests-for-android%2F)
    
- [Shot](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fpedrovgs%2FShot)
    

这里主要介绍Shot，因为最新版本中它已经支持了Compose而且使用和配置更加简单。

- 添加依赖和plugin `classpath 'com.karumi:shot:<LATEST_RELEASE>'`
    
- 修改需要UI测试module的build.gradle

```kotlin
apply plugin: 'shot'

android {
  // ...
  defaultConfig {
      testInstrumentationRunner "com.karumi.shot.ShotTestRunner"
  }
  // ...
```

- 对Activity或者Compose截图

```kotlin
class MyActivityTest: ScreenshotTest {
      @Test
      fun theActivityIsShownProperly() {
          val activity = ActivityScenario.launch(MainActivity::class.java)
          compareScreenshot(activity)
      }

      @Test
      fun rendersGreetingMessageForTheSpecifiedPerson() {
          composeRule.setContent { Greeting(greeting) }
          compareScreenshot(composeRule)
      }
  }
```

- 创建一个Android模拟器并且启动，UI测试会运行在真实的设备或者模拟器上面
    
- 运行命令获得基准的UI截图测试结果
```bash
./gradlew <Flavor><BuildType>ExecuteScreenshotTests -Precord 
或者
./gradlew executeScreenshotTests -Precord
```
这时候我们会看到设备上会自动执行写好的代码行为，例如点击按钮或者滑动界面，并且在执行完毕之后我们可以在AS的项目看到很多截图文件`app/screenshots/debug/***.png`

- 验证UI截图测试
```bash
./gradlew <Flavor><BuildType>ExecuteScreenshotTests
或者
./gradlew executeScreenshotTests
```

- 控制台输出测试报告结果

```bash
> Task :app:debugExecuteScreenshotTests
  ✅  Yeah!!! Your tests are passing.
  🤓  You can review the execution report here: /Users/xxx/app/build/reports/shot/debug/verification/index.html

  BUILD SUCCESSFUL in 6m 18s
```

- 假如有人把逻辑改错了，我们在浏览器打开文件路径会看到这样的报告
- 如果遇到动画引起的测试错误，可以创建一个命令执行文件`verify_screenshots.sh`，粘贴如下代码。用的时候控制台执行`./verify_screenshots.sh`即可
```bash
set -e
  adb devices
  adb shell settings put global window_animation_scale 0
  adb shell settings put global transition_animation_scale 0
  adb shell settings put global animator_duration_scale 0

  ./gradlew executeScreenshotTests

  adb shell settings put global window_animation_scale 1
  adb shell settings put global transition_animation_scale 1
  adb shell settings put global animator_duration_scale 1
```

## 结语

此篇是MVI架构实践的最后一篇和MVI架构关系不是很大，主要是商业化项目质量管理的介绍和实战应用。对于中小型项目我们并不需要UI测试来帮助质量管理，如果是大型项目是开发者必须要参与和学习掌握的技能。由于这个系列的很多代码和公司的技术保密要求有关系，我会对代码进行逐步排查和梳理后续补充在文档中供大家参考学习。



# 原文链接
[Android的MVI架构最佳实践(五)：UI测试](https://hoooopy.com/index.php/2023/10/25/android%e7%9a%84mvi%e6%9e%b6%e6%9e%84%e6%9c%80%e4%bd%b3%e5%ae%9e%e8%b7%b5%e4%ba%94%ef%bc%9aui%e6%b5%8b%e8%af%95/)
