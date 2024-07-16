---
author: zjmantou
title: Gradle
time: 2023-11-07 周二
tags:
  - Android
  - 技术
  - Gradle
---
# 自动打包

```groovy
android {

    ...

    signingConfigs {
        sing {
            storeFile file('your.jks')
            storePassword 'storePassword'
            keyAlias = 'keyAlias'
            keyPassword 'keyPassword'
        }
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.sing
        }
        debug {
            signingConfig signingConfigs.sing
        }
    }

    ...

 }   

```

Gradle—>app—>build—>assemble+渠道名—>双击Run

## 自定义输出路径

1. 文件名中不能出现/字符，否则会被分割成文件名。
2. 文件名中不能包含一些特殊字符如冒号（中文英文冒号都不行），编译会报错。

```groovy

android {
	...
applicationVariants.all { variant ->
        //release包才执行
        if (variant.name != "release") return
        variant.outputs.all() { output ->
            def outputFile = output.outputFile
            if (outputFile != null && outputFile.name.endsWith('.apk')) {
                //打包时间 yyyy-MM-dd HH：mm
                def formattedDate = new Date().format('MM-dd_HH.mm')
                // 自定义文件名
                outputFileName = "App-${variant.flavorName}-${variant.buildType.name}_v${defaultConfig.versionName}(${formattedDate}).apk"
                // 自定义输出路径
                variant.getPackageApplication().outputDirectory = new File(rootDir.absolutePath + "/app/apks")
            }
        }
    }
}



```


# build.gradle配置说明

在android{}模块中可以包含以下直接配置项：

- defaultConfig{} 默认配置，是ProductFlavor类型。它共享给其他ProductFlavor使用
- sourceSets{ } 源文件目录设置，是AndroidSourceSet类型。
- buildTypes{ } BuildType类型
- signingConfigs{ } 签名配置，SigningConfig类型
- productFlavors{ } 产品风格配置，ProductFlavor类型
- testOptions{ } 测试配置，TestOptions类型
- aaptOptions{ } aapt配置，AaptOptions类型
- lintOptions{ } lint配置，LintOptions类型
- dexOptions{ } dex配置，DexOptions类型
- compileOptions{ } 编译配置，CompileOptions类型
- packagingOptions{ } PackagingOptions类型
- jacoco{ } JacocoExtension类型。 用于设定 jacoco版本
- splits{ } Splits类型

# 增加编译速度

## 开启守护进程

org.gradle.daemon=true

## 配置gradle.properties

```groovy
# 开启线程守护，第一次编译时开线程，之后就不会再开了
org.gradle.daemon=true

# Try and findout the best heap size for your project build.
// 配置编译时的虚拟机大小
org.gradle.jvmargs=-Xmx5120m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8

# Modularise your project and enable parallel build
// 开启并行编译，相当于多条线程再走
org.gradle.parallel=true

# Enable configure on demand.
//启用新的孵化模式
org.gradle.configureondemand=true

```

## 去掉耗时任务（比如lint）

通过`gradle build -profile`查看哪个比较耗时任务


# Gradle生命周期

Gradle的生命周期包括三个阶段：初始化、配置和执行。

1. 初始化阶段：这个阶段会解析整个工程中所有的Project，构建所有的Project对应的Project对象。在初始化阶段和配置阶段之间的监听点是beforeEvaluate，配置阶段执行之前。
2. 配置阶段：这个阶段会解析所有的projects对象中的task，构建好所有task的拓扑图。在配置阶段之后，执行阶段之前监听点是afterEvaluate。
3. 执行阶段：这个阶段会执行具体的task及其依赖task。

# 参考链接

[Android Gradle实现一键签名打包](https://blog.csdn.net/DeMonliuhui/article/details/105710014)


[Android Studio使用Gradle实现自动打包，签名，自定义apk文件名，多渠道打包，集成系统签名证书【附效果图附源码】](https://blog.csdn.net/lucherr/article/details/72436018)


[Google I/O 中提到的提高 Android studio 的编译速度的几个建议](https://juejin.cn/post/6844903481917046791)

