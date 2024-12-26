---
author: zjmantou
title: Espresso
time: 2024-08-28 周三
tags:
  - Android
  - 单元测试
  - Espresso
---

[官方文档](https://developer.android.com/training/testing/espresso?hl=zh-cn) 

主要用来进行Android的界面测试 

# 比较好的Blog 

- [Android 自动化测试 Espresso篇：简介&基础使用](https://blog.csdn.net/mq2553299/article/details/74067002) 
- [全副武装！AndroidUI自动化测试在RxImagePicker中的实践历程](https://blog.csdn.net/mq2553299/article/details/81294514) 
- [Android 自动化测试 Espresso篇：异步代码测试](https://blog.csdn.net/mq2553299/article/details/74490718) 

# 软件包

- `espresso-core` - 包含核心和基本的 `View` 匹配器、操作和 断言。请参阅 [基本信息](https://developer.android.com/training/testing/espresso/basics?hl=zh-cn) 和[食谱](https://developer.android.com/training/testing/espresso/recipes?hl=zh-cn)。
- [`espresso-web`](https://developer.android.com/training/testing/espresso/web?hl=zh-cn) - 包含 `WebView` 支持的资源。
- [`espresso-idling-resource`](https://developer.android.com/training/testing/espresso/idling-resource?hl=zh-cn) - Espresso 与后台作业同步的机制。
- `espresso-contrib` - 包含 `DatePicker` 的外部贡献， `RecyclerView` 和 `Drawer` 操作、无障碍功能检查以及 `CountingIdlingResource`。
- [`espresso-intents`](https://developer.android.com/training/testing/espresso/intents?hl=zh-cn) - 用于对封闭测试的 intent 进行验证和打桩的扩展。
- `espresso-remote` - Espresso 的[多进程](https://developer.android.com/training/testing/espresso/multiprocess?hl=zh-cn)功能的位置。 
- 
# 示例 

- [Espresso 代码示例](https://github.com/googlesamples/android-testing) 包括各种 Espresso 示例。
- [BasicSample](https://github.com/android/testing-samples/tree/main/ui/espresso/BasicSample)： 基本的 Espresso 示例。

- [CustomMatcherSample](https://github.com/android/testing-samples/tree/main/ui/espresso/CustomMatcherSample)： 介绍如何扩展 Espresso 以与 `EditText` 对象的 hint 属性匹配。
- [RecyclerViewSample](https://github.com/android/testing-samples/tree/main/ui/espresso/RecyclerViewSample)： 适用于 Espresso 的 `RecyclerView` 操作。

- [IntentsBasicSample](https://github.com/android/testing-samples/tree/main/ui/espresso/IntentsBasicSample)：`intended()` 和 `intending()` 的基本用法。
- [IdlingResourceSample](https://github.com/android/testing-samples/tree/main/ui/espresso/IdlingResourceSample)：与后台作业同步。
- [BasicSample](https://github.com/android/testing-samples/tree/main/ui/espresso/BasicSample)：基本的 Espresso 示例。
- [CustomMatcherSample](https://github.com/android/testing-samples/tree/main/ui/espresso/CustomMatcherSample)：展示如何扩展 Espresso 以与 `EditText` 对象的 hint 属性匹配。
- [DataAdapterSample](https://github.com/android/testing-samples/tree/main/ui/espresso/DataAdapterSample)：展示 Espresso 中适用于列表和 `AdapterView` 对象的 onData() 入口点。
- [IntentsAdvancedSample](https://github.com/android/testing-samples/tree/main/ui/espresso/IntentsAdvancedSample)：模拟用户使用相机获取位图。
- [MultiWindowSample](https://github.com/android/testing-samples/tree/main/ui/espresso/MultiWindowSample)：展示了如何将 Espresso 指向不同的窗口。
- [RecyclerViewSample](https://github.com/android/testing-samples/tree/main/ui/espresso/RecyclerViewSample)：Espresso 的 `RecyclerView` 操作。
- [WebBasicSample](https://github.com/android/testing-samples/tree/main/ui/espresso/WebBasicSample)：使用 Espresso-Web 与 `WebView` 对象交互。
- [BasicSampleBundled](https://github.com/android/testing-samples/tree/main/ui/espresso/BasicSampleBundled)：Eclipse 和其他 IDE 的基本示例。

# 环境设置 

避免不稳定，建议关掉开发者选项中的几个设置：
- 窗口动画缩放
- 过渡动画缩放
- Animator 时长缩放 

# 禁用上传分析数据 

```shell
adb shell am instrument **-e disableAnalytics true**
```

# 从命令行启动测试 

```shell
./gradlew connectedAndroidTest
```

![Espresso备忘录](https://developer.android.com/static/images/training/testing/espresso-cheatsheet.png?hl=zh-cn)

# [空闲资源](https://juejin.cn/post/6844904181111734279#heading-14)  

空闲资源表示结果会影响界面测试中后续操作的异步操作。通过向 `Espresso` 注册空闲资源，可以在测试应用时更可靠地验证这些异步操作。

## 官方demo 

- [IdlingResourceSample](https://github.com/android/testing-samples/tree/main/ui/espresso/IdlingResourceSample)： 与后台作业同步。 

# Espresso-Intents 

支持对被测试应用发出的Intent进行验证和打桩。 

## 官方Dmeo 

- [IntentsBasicSample](https://github.com/android/testing-samples/tree/main/ui/espresso/IntentsBasicSample)： `intended()` 和 `intending()` 的基本用法。
- [IntentsAdvancedSample](https://github.com/android/testing-samples/tree/main/ui/espresso/IntentsAdvancedSample) 实例获取数据： 模拟用户使用相机获取位图。

# 列表 

滚动到特定项目或对特定项目执行操作 列表类型：适配器视图和 recycler 视图。 

## Demo 

- [DataAdapterSample](https://github.com/android/testing-samples/tree/main/ui/espresso/DataAdapterSample)： 展示 Espresso 中适用于列表和 `AdapterView` 的 `onData()` 入口点 对象的操作。 

# 多进程测试 

espresso-remote 

# 一些测试方案

- 匹配View旁边的View：hasSibling
- 操作栏内的View 
- 断言View未显示：isDisplayed
- 断言View不存在：doesNotExist()
- 断言数据项不在适配器中
```kotlin
private fun withAdaptedData(dataMatcher: Matcher<Any>): Matcher<View> {
    return object : TypeSafeMatcher<View>() {

        override fun describeTo(description: Description) {
            description.appendText("with class name: ")
            dataMatcher.describeTo(description)
        }

        public override fun matchesSafely(view: View) : Boolean {
            if (view !is AdapterView<*>) {
                return false
            }

            val adapter = view.adapter
            for (i in 0 until adapter.count) {
                if (dataMatcher.matches(adapter.getItem(i))) {
                    return true
                }
            }

            return false
        }
    }
}

fun testDataItemNotInAdapter() {
    onView(withId(R.id.list))
          .check(matches(not(withAdaptedData(withItemContent("item: 168")))))
    }
}
```

- 自定义故障处理程序 
```kotlin
private class CustomFailureHandler(targetContext: Context) : FailureHandler {
    private val delegate: FailureHandler

    init {
        delegate = DefaultFailureHandler(targetContext)
    }

    override fun handle(error: Throwable, viewMatcher: Matcher<View>) {
        try {
            delegate.handle(error, viewMatcher)
        } catch (e: NoMatchingViewException) {
            throw MySpecialException(e)
        }

    }
}

@Throws(Exception::class)
override fun setUp() {
    super.setUp()
    getActivity()
    setFailureHandler(CustomFailureHandler(
            ApplicationProvider.getApplicationContext<Context>()))
}
```

- 非默认窗口为目标 
```kotlin
onView(withText("South China Sea"))
    .inRoot(withDecorView(not(`is`(getActivity().getWindow().getDecorView()))))
    .perform(click())
```

- 匹配列表中的Footer和Header 

具体参考链接：[Espresso测试方案](https://developer.android.com/training/testing/espresso/recipes?hl=zh-cn) 

# Espresso Web 

测试WebView 



