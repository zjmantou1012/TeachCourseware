---
author: zjmantou
title: 00-gradle-recipes笔记
time: 2024-11-23 周六
tags:
  - Gradle
  - 笔记
---
[github demo](https://github.com/android/gradle-recipes)

gradle版本：8.2 

# Theme 

## Assets 

### addGeneratedSourceFolder 

添加源文件并写入内容 

### legacyTaskBridging 

拷贝文件内容 

## Manifest 

### transformManifest 

将合并后的menifest文件内容做修改 

### createSingleArtifact 

创建单个的Manifest文件 

### perVariantManifestPlaceholder 

给manifest文件中的manifestPlaceholders赋值。 

## Arifact API

### addMultipleArtifact 

将静态目录添加到MultipleArtifact实例中。

### transformAllClasses 

处理、转换`.dex`文件类. 

### getScopedArtifacts 

在Scope内为每个变体添加检查任务 

### transformDirectory 

使用`InAndOutDirectoryOperationRequest.toTransform()` 方法对文件进行内容转换并输出 

### workerEnabledTransformation 

`InAndOutDirectoryOperationRequest.toTransformMany()`  

并行任务，将任务划分为更小的、独立的工作单元. 

将apk所在的目录拷贝到output。 

### appendToScopedArtifacts 

通过`ScopedArtifactsOperation.toAppend()`  在指定Scope中添加文件。 



