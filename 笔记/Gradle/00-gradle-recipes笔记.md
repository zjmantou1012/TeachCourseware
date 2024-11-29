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

# 基本概念 

## variant 

变体 gradle构建的变体 

## artifacts 

通过其提供的api可以访问Variant对象上产生的artifact（工件），artifact是有AGP在构建过程中产生的临时或最终文件或目录。

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

## Artifact API 

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

### getMultipleArtifact 

对MULTIDEX_KEEP_PROGUARD输入的文件进行操作 

### getSingleArtifact  

对每个变体操作，本示例使用SingleArtifact.BUNDLE，检查是否有appbundle的.aab文件 

### appendToMultipleArtifact 

创建文件添加到MutipleArtifact， 本节演示将文件添加到NATIVE_DEBUG_METADATA目录下。 

## DSL 
### extendingAgp 

第三方插件扩展Android DSL块 （待阅读）

### addBuildTypeUsingDslFinalize 

在执行build之后，创建variants之前，扩展自定义的DSL。该示例通过对finalizeDsl添加一个自定义的buildType。 

## Dependency Resolution 依赖解析 

### variantDependencySubstitutionTest 

使用variant API 访问某个特定variant（比如buildType为release的variant）的编译和运行配置，替换其依赖关系。 

## Source 

[[#Assets#addGeneratedSourceFolder]]

### addCustomSourceType 

将新的source file添加到新建的source type中，可以被自定义的Variant API毁掉插件和任务使用。 

该示例中使用了SourceDirectories.Flat.addStaticSourceDirectory 和 addGeneratedSourceDirectory 分别创建了srcDIr和generated目录。 


[[#Assets#legacyTaskBridging]]


