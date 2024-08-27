---
author: zjmantou
title: Benchmark
time: 2024-08-27 周二
tags:
  - Android
  - 单元测试
  - Benchmark
---
# Macrobenchmark 

适合测试应用的较大用力，包括应用启动、复杂的界面操作（比如滚动，动画运行）

```kotlin
@RunWith(AndroidJUnit4::class)  
class ExampleStartupBenchmark {  
    @get:Rule  
    val benchmarkRule = MacrobenchmarkRule()  
  
    @Test  
    fun startup() = benchmarkRule.measureRepeated(  
        packageName = "com.example.kotlinproject",  
        metrics = listOf(StartupTimingMetric()),  
        iterations = 5,  
        startupMode = StartupMode.COLD  
    ) {  
        pressHome()  
        startActivityAndWait()  
    }  
}
```

## 指标 

- [`StartupTimingMetric`](https://developer.android.com/reference/kotlin/androidx/benchmark/macro/StartupTimingMetric?hl=zh-cn)
- [`FrameTimingMetric`](https://developer.android.com/reference/kotlin/androidx/benchmark/macro/FrameTimingMetric?hl=zh-cn)
- [`TraceSectionMetric`](https://developer.android.com/reference/kotlin/androidx/benchmark/macro/TraceSectionMetric?hl=zh-cn) 

[捕获 Macrobenchmark 指标](https://developer.android.com/topic/performance/benchmarking/macrobenchmark-metrics?hl=zh-cn)。 

```kotlin
benchmarkRule.measureRepeated(
    packageName = TARGET_PACKAGE,
    **metrics = listOf(
        FrameTimingMetric(),
        TraceSectionMetric("RV CreateView"),
        TraceSectionMetric("RV OnBindView"),
    ),**
    iterations = 5,
    // ...
)
```

这是一段追踪RecycleView中`createViewHolder` 和 `bindViewHolder` 的指标。 

### StartupTimingMetric 

- `timeToInitialDisplayMs`：从系统收到启动Intent到渲染目标的Activity的第一帧所用的时间。 
- `timeToFullDisplayMs`： 从系统收到启动intent到应用使用`reportFullyDrawn()`方法报告完全绘制所用的时间。 
### FrameTimingMetric 

捕获设定的基准测试所生成帧（比如滚动和动画）的时间信息并输出：
- `frameOverrunMs`：这个帧完全呈现用时与呈现的时限差。正数表示出现丢帧和可见的卡顿，负数表示用时在时限内。（12以上版本）
- `frameDurationCpuMs`：通过洁面县城和`RenderThread` 在CPU中生成帧所用的时间。 

### TraceSectionMetric 

捕获指定输入的Trace的耗时。 

### PowerMetric 

捕获电源类别在测试期间的耗电量和能量变化。


## CompilationModel 

预编译模式，可以定义预编译的量，定义将应用的多大部分从DEX字节码预编译为机器码。 

- DEFAULT 
- Full 
- Partial 
- None 
- Ignore 

## StartupMode 

启动模式： `COLD`、`WARM`、`HOT` 


## 控制应用 

使用资源 ID 查找 RecyclerView，然后向下滚动几次： 

```kotlin 

@Test
fun scrollList() {
    benchmarkRule.measureRepeated(
        // ...
        setupBlock = {
            // Before starting to measure, navigate to the UI to be measured
            val intent = Intent("$packageName.RECYCLER_VIEW_ACTIVITY")
            startActivityAndWait(intent)
        }
    ) {
        val recycler = device.findObject(By.res(packageName, "recycler"))
        // Set gesture margin to avoid triggering gesture navigation
        // with input events from automation.
        recycler.setGestureMargin(device.displayWidth / 5)

        // Scroll down several times
        repeat(3) { recycler.fling(Direction.DOWN) }
    }
}
```


控制内部板块，点击按钮： 

```kotlin
@Test
fun nonExportedActivityScrollList() {
    benchmarkRule.measureRepeated(
        // ...
        setupBlock = setupBenchmark()
    ) {
        // ...
    }
}

private fun setupBenchmark(): MacrobenchmarkScope.() -> Unit = {
    // Before starting to measure, navigate to the UI to be measured
    startActivityAndWait()

    // click a button to launch the target activity.
    // While we use button text  here to find the button, you could also use
    // accessibility info or resourceId.
    val selector = By.text("RecyclerView")
    if (!device.wait(Until.hasObject(selector), 5_500)) {
        fail("Could not find resource in time")
    }
    val launchRecyclerActivity = device.findObject(selector)
    launchRecyclerActivity.click()

    // wait until the activity is shown
    device.wait(
        Until.hasObject(By.clazz("$packageName.NonExportedRecyclerActivity")),
        TimeUnit.SECONDS.toMillis(10)
    )
}
```

## 插桩参数 

### 使用方式 

Gradle： 

```groovy
android {
    defaultConfig {
        // ...
        testInstrumentationRunnerArguments["androidx.benchmark.dryRunMode.enable"] = "true"
    }
}
```

配置中增加 

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202408271657941.png)

命令行： 

```shell
./gradlew :benchmark:connectedAndroidTest -P android.testInstrumentationRunnerArguments.androidx.benchmark.enabledRules=BaselineProfile
```

### androidx.benchmark.compilation.enabled 

每次迭代之间停用编译，默认值：true 

### androidx.benchmark.dryRunMode.enable 

单个循环中运行基准测试，以验证基准测试是否正常运行，默认值：false 

### androidx.benchmark.enabledRules 

允许仅过滤一种类型的测试：基准配置文件生成或 Macrobenchmark 测试。还支持逗号分隔列表。 

- Macrobenchmark
- BaselineProfile
默认值：未指定 

### androidx.benchmark.junit4.SideEffectRunListener 

在基准测试期间停用后台工作 

将listener参数设置为这个，默认为未指定。 

### androidx.benchmark.fullTracing.enable 

启用 androidx.tracing.perfetto 跟踪点 

默认值：false 

### androidx.benchmark.profiling.mode 

允许在运行基准测试时捕获轨迹文件。 

- `MethodTracing`
- `StackSampling`
- `None` 

### androidx.benchmark.startupProfiles.enable 

允许您在基准测试期间禁止生成启动配置文件 

默认值：true 

### androidx.benchmark.suppressErrors 

接受以逗号分隔的错误列表，以供转换为警告。 

- 可用选项：
    
    - `DEBUGGABLE`
        
        `DEBUGGABLE` 错误表示目标软件包是在其清单中设置了 `debuggable=true` 的情况下运行的，这会大幅降低运行时性能以支持调试功能。为避免此错误，请使用 `debuggable=false` 运行基准测试。可调试的参数会对执行速度产生影响，也就是说，基准测试改进可能不会反映到真实用户的体验中，或者可能会降低发布性能。
        
    - `LOW-BATTERY`
        
        当电池电量不足时，设备通常会降低性能以节省电量，例如停用大核心。即使设备已接通电源，也会发生这种情况。只有在您有意降低应用性能的情况下，才应忽略此错误。
        
    - `EMULATOR`
        
        `EMULATOR` 错误表示基准测试是在模拟器上运行，并不代表真实的用户设备。模拟器基准测试改进可能不会反映到用户的实际体验中，或者可能会降低设备的实际性能。您应改用实体设备来进行基准测试。忽略此错误时需要慎之又慎。
        
    - `NOT-PROFILEABLE`
        
        目标软件包 `$packageName` 在未设置 `<profileable shell=true>` 的情况下运行。Android 10 和 11 中要求启用配置文件功能，以便 Macrobenchmark 从目标进程中捕获详细的跟踪信息，例如应用或库中定义的系统跟踪部分。忽略此错误时需要慎之又慎。
        
    - `METHOD-TRACING-ENABLED`
        
        针对要进行基准测试的应用运行的 Macrobenchmark 已启用方法跟踪。这会减慢虚拟机的运行速度，因此请仅考虑跟踪文件中的相对指标，例如比较首次运行与第二次运行的速度。如果您在具有不同方法追踪选项的 build 之间比较基准测试，则忽略此错误可能会导致结果不准确。
        
- **默认为**：一个空列表 

### additionalTestOutputDir 

配置在设备上保存 JSON 基准测试报告和性能分析结果的位置。


# Microbenchmark 

适合应用中频繁运行的CPU工作。 

比如一次显示一项的RecyclerView滚动，数据转换或处理以及反复使用的其他代码段。 

## 参考资料 

demo： 
- [性能示例](https://github.com/android/performance-samples)
- [PagingWithNetworkSample](https://github.com/android/architecture-components-samples/tree/main/PagingWithNetworkSample/benchmark)
- [WorkManagerSample](https://github.com/android/architecture-components-samples/tree/main/WorkManagerSample/benchmark)

Blog：
- [在 CI 中使用基准测试应对回归问题](https://medium.com/androiddevelopers/fighting-regressions-with-benchmarks-in-ci-6ea9a14b5c71)

![项目结构](https://developer.android.com/static/topic/performance/images/benchmark_images/microbenchmark_modules.png?hl=zh-cn) 

