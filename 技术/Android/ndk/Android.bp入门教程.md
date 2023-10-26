---
author: zjmantou
title: Android.bp入门教程
time: 2023-09-18 周一
tags:
  - Android
---
## Soong 编译系统

在 Android 7.0 发布之前，Android 仅使用 [GNU Make](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.gnu.org%2Fsoftware%2Fmake%2F) 描述和执行其构建规则。Make 构建系统得到了广泛的支持和使用，但在 Android 层面变得缓慢、容易出错、无法扩展且难以测试。[Soong 构建系统](https://links.jianshu.com/go?to=https%3A%2F%2Fandroid.googlesource.com%2Fplatform%2Fbuild%2Fsoong%2F%2B%2Frefs%2Fheads%2Fmaster%2FREADME.md)正好提供了 Android build 所需的灵活性。

Soong 构建系统是在 Android 7.0 (Nougat) 中引入的，旨在取代 Make。它利用 [Kati](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fgoogle%2Fkati%2Fblob%2Fmaster%2FREADME.md) GNU Make 克隆工具和 [Ninja](https://links.jianshu.com/go?to=https%3A%2F%2Fninja-build.org%2F) 构建系统组件来加速 Android 的构建。

## Android.bp文件格式

Android.bp的语法在设计上要比Android.mk简单一些，主要是因为Android.bp没有条件或控制流语句。

注意：Android.bp不支持条件语句！在实际项目中，如果构建的脚本必须包含条件语句，建议使用Android.mk或使用Go语言

### 模块与示例

Android.bp 文件中的模块以`模块类型`开头，然后是一组格式属性：`name: value`，在一点上Android.bp的语法结构与JSON的语法结构相似，都是以键值对的形式编写。下面是一个简单示例：

```
android_app {
	name: "Provision",     
	srcs: ["**/*.java"],     
	platform_apis: true,     
	product_specific: true,     
	certificate: "platform", 
} 
```

每个模块都必须具有 `name` 属性，并且相应值在所有 `name` 文件中必须是唯一的，仅有两个例外情况是命名空间和预构建模块中的 `Android.bp` 属性值，这两个值可能会重复。srcs 属性以字符串列表的形式指定用于构建模块的源文件。可以使用模块引用语法 `":<module-name>"` 来引用生成源文件的其他模块的输出，如 `genrule` 或 `filegroup`。

```
android_app{} 
```

表示该模块用于构建一个apk

```
name: "Provision", 
```

如果指定了 `android_app` 模块类型（在代码块的开头），就需要设定该模块的name 。此设置会为模块命名，生成的 APK 将与模块名称相同，不过带有 .apk 后缀。例如，在本例中，生成的 APK 将命名为 Provision.apk

```
srcs: ["**/*.java"], 
```

`srcs`用于指定当前的模块编译的源码位置，*表示通配符

```
platform_apis: true, 
```

设置该标记后会使用sdk的hide的api來编译。编译的APK中使用了系统级API，必须设定该值。和Android.mk中的`LOCAL_PRIVATE_PLATFORM_APIS`的作用相同

```
product_specific: true, 
```

设置该标记后，生成的apk会被安装在Android系统的product分区，和Android.mk中`LOCAL_PRODUCT_MODULE`作用相同

```
certificate: "platform", 
```

`certificate` 用于指定APK的签名方式。如果不指定，默认使用testkey签名。与`LOCAL_CERTIFICATE`作用相同。

Android中共有四中签名方式：

- testkey：普通APK，默认使用该签名。
- platform：该APK完成一些系统的核心功能。经过对系统中存在的文件夹的访问测试，这种方式编译出来的APK所在进程的UID为system。
- shared：该APK需要和home/contacts进程共享数据。
- media：该APK是media/download系统中的一环。

### 常见模块类型

在Android.bp中我们会基于模块类型来构建我们所需要的东西，常用的有以下几种类型

#### android_app

用于构建Android 应用程序安装包，是Android系统应用开发中常用的模块类型，与Android.mk中的`BUILD_PACKAGE`作用相同。

android_app_certificate

`android_app_certificate`模块可由`android_app`模块的证书属性`certificate`引用以选择签名密钥。

#### java_library

`java_library`用于将源代码构建并链接到设备的`.jar`文件中。默认情况下，`java_library`只有一个变量，它生成一个包含根据设备引导类路径编译的`.class`文件的`.jar`包。生成的jar不适合直接安装在设备上，通过会用作另一个模块的`static_libs`依赖项。

如果指定“installable:true”将生成一个包含“classes.dex”文件的“.jar”文件，适合在设备上安装。指定'host_supported:true'将产生两个变量，一个根据device的bootclasspath编译，另一个根据host的bootclasspath编译。

#### java_library_static

`java_library_static`作用等同于`java_library`，但是`java_library_static`已经过时，不推荐使用

#### android_library

`android_library`将源代码与android资源文件一起构建并链接到设备的“.jar”文件中。android_library有一个单独的变体，它生成一个包含根据device的bootclasspath编译的`.class`文件的`.jar`文件，以及一个包含使用aapt2编译的android资源的`.package-res.apk`文件。生成的apk文件不能直接安装在设备上，但可以用作`android_app`模块的`static_libs`依赖项。

#### cc_library

`cc_library`为device或host创建静态库或共享库。默认情况下，`cc_library`具有针对设备的单一变体。指定'host_supported:true'还会创建一个以主机为目标的库。与cc_library相关的模块类型还有`cc_library_shared`、`cc_library_headers`、`cc_library_static`等。

Android.bp中涉及到的模块类型非常的多，我们可以在[Soong模块和属性列表](https://links.jianshu.com/go?to=https%3A%2F%2Fci.android.com%2Fbuilds%2Fsubmitted%2F7621700%2Flinux%2Flatest%2Fview%2Fsoong_build.html)中查看其它的模块类型以及用法。

### 设置变量

在bp中可以通过=号来设定一个全局变量

```
src_path = ["**/*.java"] 
android_app {     
	name: "Provision",     
	srcs: src_path,     
	platform_apis: true,     
	product_specific: true,     
	certificate: "platform", 
} 
```

### 默认模块

soong提供了一系列xx_defaults模块类型，例如：`cc_defaults`, `java_defaults`, `doc_defaults`, `stubs_defaults`等等。

`xx_defaults`的模块提供了一组可由其它模块继承的属性。其它模块可以通过defaults：["<：default_module_name>"]来继承xx_defaults类型模块中定义的属性。xxx_defaults类型的模块可以被多个模块继承，减少我们在bp中书写重复的属性。

```
cc_defaults {     
	name: "gzip_defaults",     
	shared_libs: ["libz"],     
	stl: "none", 
} 

cc_binary {     
	name: "gzip",     
	defaults: ["gzip_defaults"],     
	srcs: ["src/test/minigzip.c"], 
} 
```

### 数据类型

Android.bp中变量和属性是强类型，变量根据第一项赋值动态变化，属性由模块类型静态设置。支持的类型为：

- 布尔值（`true`或 `false`）
- 整数 （`int`）
- 字符串（"`string`"）
- 字符串列表 (["`string1`", "`string2`"])
- 映射 ({`key1`: "`value1`", `key2`: ["`value2`"]})

映射可以包含任何类型的值，包括嵌套映射。

### 条件语句

Soong 不支持 `Android.bp` 文件中的条件语句。编译规则中需要条件语句的复杂问题将在 Go（在这种语言中，您可以使用高级语言功能，并且可以跟踪条件语句引入的隐式依赖项）中处理。大多数条件语句都会转换为映射属性，其中选择了映射中的某个值并将其附加到顶级属性。

例如，要支持特定于架构的文件，可以使用以下命令：

```
cc_library {   
	...   
	srcs: ["generic.cpp"],   
	arch: {     
		arm: {       
			srcs: ["arm.cpp"],     
		},     
		x86: {       
		srcs: ["x86.cpp"],     
		},   
	}, 
}
```

注意：Android.bp不支持条件语句！在实际项目中，如果构建的脚本必须包含条件语句，建议使用Android.mk

### 运算符

可以使用 + 运算符附加字符串、字符串列表和映射。可以使用 + 运算符对整数求和。附加映射会生成两个映射中键的并集，并附加在两个映射中都存在的所有键的值。

### 注释

Android.bp文件可以和C/C++的注释类似，可以包含多行注释和单行注释。

1. 多行注释：/ * 注释 * /
2. 单行注释：//注释。

关于Android.bp还有一些其它的基础知识，请继续参阅[Soong 编译系统](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fsetup%2Fbuild)

## 使用示例

关于Android.bp如何使用，我们可以参考系统中已有模块的是如何编写的。

该Android.bp位于Android 10 : packages/apps/Car/Notification 下

```
// 构建可执行程序 
android_app {     
	// 设定可执行的程序的名称，编译后会生成一个 CarNotification.apk     
	name: "CarNotification",     
	// 指定java源码的位置     
	srcs: ["src/**/*.java"],     
	// 指定资源文件的位置     
	resource_dirs: ["res"],     
	// 允许使用系统hide api     
	platform_apis: true,     
	// 设定apk签名为 platform     
	certificate: "platform",     
	// 设定apk安装路径为priv-app     
	privileged: true,     
	// 是否启用代码优化，android_app中默认为true，java_library中默认为false     
	optimize: {         
		enabled: false,     
	},     
	// 是否预先生成dex文件，默认为true。该属性会影响应用的首次启动速度以及Android系统的启动速度     
	dex_preopt: {         
		enabled: false,     
	},     
	// 引入java静态库     
	static_libs: [         
		"androidx.cardview_cardview",         
		"androidx.recyclerview_recyclerview",
		"androidx.palette_palette",         
		"car-assist-client-lib",         
		"android.car.userlib",         
		"androidx-constraintlayout_constraintlayout"     
	],     
	// 引入java库     
	libs: ["android.car"],     
	product_variables: {         
		pdk: {             
			enabled: false,         
		},     
	},     
	// 设定依赖模块。如果安装了此模块，则要还需要安装的其他模块的名称     
	required: ["privapp_whitelist_com.android.car.notification"] 
} 
// As Lib 
android_library {     
	name: "CarNotificationLib",     
	srcs: ["src/**/*.java"],     
	resource_dirs: ["res"],     
	manifest: "AndroidManifest-withoutActivity.xml",     
	platform_apis: true,     
	optimize: {         
		enabled: false,     
	},     
	dex_preopt: {         
		enabled: false,     
	},     
	static_libs: [         
		"androidx.cardview_cardview",         
		"androidx.recyclerview_recyclerview",         
		"androidx.palette_palette",         
		"car-assist-client-lib",         
		"android.car.userlib",         
		"androidx-constraintlayout_constraintlayout"     
	],     
	libs: ["android.car"],     
	product_variables: {         
		pdk: {             
			enabled: false,         
		},     
	}, 
}
```

## 常见疑问

### Android.bp引用Android.mk编译的模块

Android.bp编译的模块无法访问到Android.mk编译的模块！相反，Android.mk可以引用Android.bp编译的模块。

### 引入第三方jar

在实际开发中，我们经常需要在app中引入第三方的jar，在Android.bp中，可以按照下面的方式引入。在项目的根目录新建 libs文件夹，放入要导入的jar包，比如 CarServicelib.jar，然后在Android.bp中引入该jar

```
java_import {   
	name: "CarServicelib.jar",   
	jars: ["libs/CarServicelib.jar"], 
} 
android_app {     
	// 省去其它属性     
	// 引入AndroidX库下的lib     
	static_libs: [         
		"CarServicelib.jar"     
	], 
} 
```

### 引入AndroidX库

开发过程如果想要引入AndroidX的类库可以参考下面的方式编写。

```
android_app {     
	// 省去其它属性     
	// 引入AndroidX库下的lib     
	static_libs: [         
		"androidx.cardview_cardview",         
		"androidx.recyclerview_recyclerview",         
		"androidx-constraintlayout_constraintlayout"     
	], 
}
```

## 参考资料

[Soong 编译系统 | Android 开源项目 | Android Open Source Project (google.cn)](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fsetup%2Fbuild)

[Soong (googlesource.com)](https://links.jianshu.com/go?to=https%3A%2F%2Fandroid.googlesource.com%2Fplatform%2Fbuild%2Fsoong%2F%2B%2Fmaster%2FREADME.md)

[简单的 build 配置 | Android 开源项目 | Android Open Source Project](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.com%2Fcompatibility%2Ftests%2Fdevelopment%2Fblueprints)

[Android 编译之android.bp](https://www.jianshu.com/p/f69d1c381182)

  
  
作者：林栩  
链接：https://www.jianshu.com/p/f23e18933122  
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。