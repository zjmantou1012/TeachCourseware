---
author: zjmantou
title: Android 持续集成（CI）
time: 2024-08-27 周二
tags:
  - Android
  - CI/CD
  - 单元测试
---
# 官方文档 

[持续集成环境中的基准测试](https://developer.android.com/topic/performance/benchmarking/benchmarking-in-ci?hl=zh-cn) 

讲的是CI中的benchmark 

[持续集成相关知识](https://developer.android.com/training/testing/continuous-integration?hl=zh-cn)  

# 基本信息

防止在合并后破坏build。 

![CI 系统在合并之前运行检查，以保证代码库的正常运行](https://developer.android.com/static/training/testing/continuous-integration/ci1.svg?hl=zh-cn)

## 典型示例 

典型的 CI 系统遵循以下工作流或流水线：

1. CI 系统检测到代码更改，这通常是在开发者创建拉取请求时（也称为“更改列表”或“合并请求”）。
2. 它会预配和初始化服务器以运行工作流。
3. 它会根据需要提取代码以及 Android SDK 或模拟器映像等工具。
4. 它通过运行给定命令（例如 .`/gradlew build`）来构建项目。
5. 它通过运行给定命令（例如运行 .`/gradlew test`）来运行[本地测试](https://developer.android.com/training/testing/local-tests?hl=zh-cn)。
6. 它会启动模拟器并运行[插桩测试](https://developer.android.com/training/testing/instrumented-tests?hl=zh-cn)。
7. 它会上传测试结果和 APK 等工件。 

![基本CI工作流](https://developer.android.com/static/training/testing/continuous-integration/ci2.svg?hl=zh-cn) 

## CI优势 

- **提高软件质量**：CI 可以帮助您尽早发现并解决问题，从而帮助提高软件质量。这有助于减少软件版本中的 bug 数量并改善整体用户体验。
- **降低构建中断的风险**：使用 CI 实现构建流程自动化后，通过尽早解决问题，可以更好地避免构建中断的情况。
- **增强对版本的信心**：CI 有助于确保每个版本都稳定且已准备好投入生产环境。通过运行自动化测试，CI 可以发现任何潜在问题，然后再将其公开发布。
- **改善了沟通和协作**：CI 为开发者提供了一个集中分享代码和测试结果的地方，可帮助开发者和其他团队成员更轻松地协同工作并跟踪进度。
- **提高工作效率**：CI 可以自动执行那些耗时且容易出错的任务，从而提高开发者的工作效率。

# CI自动化 

## 基本作业 

- **构建**：通过从头开始构建项目，您可以确保新更改能够正确编译，并且所有库和工具彼此兼容。
- **lint 或样式检查**：这是一个可选但建议执行的步骤。当您强制执行样式规则并执行[静态分析](https://developer.android.com/studio/write/lint?hl=zh-cn)时，代码审核可以更简洁、更集中。
- **[本地测试或主机端测试](https://developer.android.com/training/testing/local-tests?hl=zh-cn)**：此类测试在执行构建的本地机器上运行。在 Android 上，这通常是 JVM，因此运行速度快且稳定。它们还包含 Robolectric 测试。

## 插桩测试 

[在模拟器或实体设备上运行的测试](https://developer.android.com/training/testing/instrumented-tests?hl=zh-cn)需要进行一些配置，等待设备启动或连接，以及执行其他会增加复杂性的操作。

您可以通过多种方式在 CI 上运行插桩测试：

- [Gradle 管理的设备](https://developer.android.com/studio/test/gradle-managed-devices?hl=zh-cn)可用于定义要使用的设备（例如“API 27 上的 Pixel 2 模拟器”），并处理设备配置。
- 大多数 CI 系统都附带用于处理 Android 模拟器的第三方插件（也称为“操作”“集成”或“步骤”）。
- 将插桩测试委托给设备场（如 [Firebase Test Lab](https://firebase.google.com/docs/test-lab?hl=zh-cn)）。设备场因具备高可靠性而设计，可以在模拟器或实体设备上运行。

## 性能回归测试 

[[Benchmark]] 

## 覆盖率回归 

[测试覆盖率](https://developer.android.com/studio/test/test-in-android-studio?hl=zh-cn#view_test_coverage)是一个指标，可帮助您和您的团队确定测试是否充分涵盖某项更改。但是，它不应是唯一的指标。通常的做法是设置在覆盖率相对于基本分支降低时失败或显示警告的回归检查。 

# [CI功能](https://developer.android.com/training/testing/continuous-integration/features?hl=zh-cn) 



