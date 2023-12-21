---
author: zjmantou
title: AndroidX迁移笔记
time: 2023-12-20 周三
tags:
  - Android
  - 笔记
  - AndroidX
---
先升级到support库最终版28.0.0，然后使用Refactor->Migrate to AndroidX选项进行迁移，期间会在gradle.properties文件下设置两个标记：
- `android.useAndroidX=true` ：Android 插件会使用对应的 AndroidX 库而非支持库。
- `android.enableJetifier=true`：Android 插件会通过重写现有第三方库的二进制文件，自动将这些库迁移为使用 AndroidX。

如果遇到问题的话参考映射表里的库和类：
- - [Maven artifact mappings](https://developer.android.com/jetpack/androidx/migrate/artifact-mappings)
- [Class mappings](https://developer.android.com/jetpack/androidx/migrate/class-mappings)

# 为什么迁移

- Android Support库已经在28版本之后不维护
- 更好的包管理。可以获得标准化和独立的版本控制，以及更好的标准化命名和更频繁的发布。
- 其他的库已经迁移到AndroidX。
- 所有新的 Jetpack 库都将在 AndroidX 命名空间中发布。因此，例如，要利用Jetpack Compose或CameraX，您需要迁移到 AndroidX 命名空间。

