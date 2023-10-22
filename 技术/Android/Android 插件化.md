---
author: zjmantou
title: Android 插件化
time: 2023-10-19 周四
tags:
  - Android
  - 技术
---
# 插桩式插件化框架

## 组件化与插件化的区别
组件化是将应用分成若干个module模块，每个模块称为一个组件，分“集成模式”和“组件模式”

组件化的弊端：
有修改就必须重新打包。

插件化将应用分成一个“宿主”模块和若干个“插件”模块。 最终打包时分开打包，“宿主”模块和“插件”模块都各自是一个单独的apk安装文件。  

组件化 主要是设计好整个程序的架构 , 使用 Gradle 控制并切换 组件模式 / 集成模式 , 核心是 组件路由 的使用 ;

**插件化 的核心就是实现 " 插件 " APK 的 动态加载与调用 **。  

## 类加载机制

![[Java类加载机制]]

## 实现原理
- 解压插件apk，拿到class.dex，自己实现一个DexClassLoader加载该dex，进而可以调用其中封装的字节码对象；
- 宿主模块中预留“代理”组件，实现“插件”组件中上下文的获取和生命周期的管理；

## 实现思路
1. 加载类对象 : 使用 DexClassLoader 加载 Activity 对应的 Class 字节码类对象 ;
2. 管理生命周期 : 处理加载进来的 Activity 类的生命周期 , 创建 ProxyActivity , 通过其生命周期回调 , 代理管理 插件包中加载的未纳入应用管理的组件 Activity 类 ;
3. 注入上下文 : 为加载进来的 Activity 类注入 上下文 ;
4. 加载资源 : 使用 AssetManager 将插件包 apk 中的资源主动加载进来 ;  

## 总结
1. 自定义一个ClassLoader，通过反射调用AssetManager方法加载资源。
2. 获取插件包中的Activity类信息。
3. 创建一个代理Activity：ProxyActivity，替换其ClassLoader和Resource。
4. 代理Activity中通过反射拿到插件Activity信息，并注入上下文，处理生命周期回调等操作。

存在的一些问题：
1. 开发需要定制：需要继承BaseActivity，与普通开发不一样；
2. 没有真正的上下文环境，调用Activity的getApplicationContext也会出问题；

### 参考链接
[Android 插件化简介 ( 组件化与插件化 )](https://hanshuliang.blog.csdn.net/article/details/117391407)
[github](https://github.com/han1202012/Plugin)


# Hook插件化框架

主要技术：
1. 反射技术；
2. 代理模式；

## Android Hook用到的动态代理技术
![[Android Hook#Android Hook主要用到的技术]]

## 插件化、静态代理
通过反射拿到Instrumentation，静态代理execStartActivity方法完成对startActivity的hook：
```Java
        // 1. 获取 Activity 字节码文件
        //    字节码文件是所有反射操作的入口
        Class<?> clazz = Activity.class;

        // 2. 获取 Activity 的 Instrumentation mInstrumentation 成员 Field 字段
        Field mInstrumentation_Field = null;
        try {
            mInstrumentation_Field = clazz.getDeclaredField("mInstrumentation");
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }

        // 3. 设置 Field mInstrumentation 字段的可访问性
        mInstrumentation_Field.setAccessible(true);


        // 4. 获取 Activity 的 Instrumentation mInstrumentation 成员对象值
        //      获取该成员的意义是 , 创建 Instrumentation 代理时, 需要将原始的 Instrumentation 传入代理对象中
        Instrumentation mInstrumentation = null;
        try {
            mInstrumentation = (Instrumentation) mInstrumentation_Field.get(this);
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }

        // 5. 将 Activity 的 Instrumentation mInstrumentation 成员变量
        //      设置为自己定义的 Instrumentation 代理对象
        try {
            mInstrumentation_Field.set(this, new InstrumentationProxy(mInstrumentation));
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }

```

代理类InstrumentationProxy:
```Java
package com.example.plugin_hook;

import android.app.Activity;
import android.app.Instrumentation;
import android.content.Context;
import android.content.Intent;
import android.os.Bundle;
import android.os.IBinder;
import android.util.Log;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class InstrumentationProxy extends Instrumentation {

    private static final String TAG = "InstrumentationProxy";
    /**
     * Activity 中原本的 Instrumentation mInstrumentation 成员
     * 从构造函数中进行初始化
     */
    final Instrumentation orginalInstrumentation;

    public InstrumentationProxy(Instrumentation orginalInstrumentation) {
        this.orginalInstrumentation = orginalInstrumentation;
    }

    public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {

        Log.i(TAG, "注入的 Hook 前执行的业务逻辑");

        // 1. 反射执行 Instrumentation orginalInstrumentation 成员的 execStartActivity 方法
        Method execStartActivity_Method = null;
        try {
            execStartActivity_Method = Instrumentation.class.getDeclaredMethod(
                    "execStartActivity",
                    Context.class,
                    IBinder.class,
                    IBinder.class,
                    Activity.class,
                    Intent.class,
                    int.class,
                    Bundle.class);
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        }

        // 2. 设置方法可访问性
        execStartActivity_Method.setAccessible(true);

        // 3. 执行 Instrumentation orginalInstrumentation 的 execStartActivity 方法
        //    使用 Object 类型对象接收反射方法执行结果
        ActivityResult activityResult = null;
        try {
            activityResult = (ActivityResult) execStartActivity_Method.invoke(orginalInstrumentation,
                    who,
                    contextThread,
                    token,
                    target,
                    intent,
                    requestCode,
                    options);
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }

        Log.i(TAG, "注入的 Hook 后执行的业务逻辑");

        return activityResult;
    }
}


```
#### 参考链接
[Hook 插件化框架 ( Hook Activity 启动过程 | 静态代理 )](https://hanshuliang.blog.csdn.net/article/details/117989555)

## 不同版本兼容问题

Hook Android内部流程时，要注意Android兼容问题； 

[Android各版本拦截进程对AMS的请求实战](https://blog.csdn.net/Androiddddd/article/details/112968387)

## Android进程相关源码 API28
[Activity 进程相关源码](https://hanshuliang.blog.csdn.net/article/details/118001126)
[主进程相关源码](https://hanshuliang.blog.csdn.net/article/details/118027393)

- 应用主进程中调用到Instrumentation中的execStartActivity，通过ams的binder调用AMS中的方法；
- AMS中最终会调用到ActivityThread的handleLaunchActivity方法，通过newActivity去创建activity实例.

## 实现原理
1. **加载插件包中的字节码**；
2. **hook 技术** : 直接通过 hook 技术, 钩住系统的 Activity 启动流程实现。
	1. Activity 对象创建之前 , 要做很多初始化操作 , 先在 ActivityRecord 中加载 Activity 信息 , 如果修改了该信息 , 将要跳转的 Activity 信息修改为插件包中的 Activity , 原来的 Activity 只用于占位 , 用于欺骗 Android 系统 ;
	2. 使用 hook 技术 , 加载插件包 apk 中的 Activity；
	3. 实现跳转的 Activity ( 插件包中的 )；
3. **资源加载 :** 主要是解决 Resources 资源冲突问题 ;

使用Hook的插件化可以不用考虑生命周期问题。  


## 总结

插件化框架主要是 通过 hook 修改 Instrumentation , 以及 劫持 ActivityManagerService ;

### Hook点选择


### 模块选择
功能比较单一 , 业务逻辑更新比较频繁 , 并且很重要的模块 , 使用插件化实现。

### 难点
Hook 插件化框架的 难点是版本兼容 , 需要逐个手动兼容 Android 低版本到最新版本 , 一旦系统更新 , 或者某厂商 ROM 更新 , 都要进行兼容测试以及改进 ;

### 解决办法
如果 Android 高版本禁止反射 @hide 方法 , 可以在 调用链上找到一个非隐藏的方法 , 总能 hook 住 ; 极端情况下 , 使用 [[动态字节码技术]] , 在运行时修改字节码数据 , 删除 @hide 注解 ;

### 参考链接

[Hook 插件化框架](https://blog.csdn.net/shulianghan/article/details/119521364)
[github链接]([https://github.com/han1202012/Plugin_Hook](https://github.com/han1202012/Plugin_Hook))


# 总结
## 占位 Activity
插件包中的 Activity 是通过正规流程 , 由 AMS 进行创建并加载的 , 但是 该 Activity 并没有在 AndroidManifest.xml 清单文件中注册 , 这里需要一个已经在清单文件注册的 Activity 欺骗系统 

## 插装式插件化 
是通过代理 Activity , 将 插件包加载的 字节码 Class 类 中 对应的 Activity 类作为一个普通的 Java 类 , 该普通的 Java 类有所有的 Activity 的业务逻辑 , 该 Activity 的生命周期 , 由代理 Activity 执行相关的生命周期方法

## hook 插件化
hook 插件化直接钩住系统中 Activity 启动流程的某个点
