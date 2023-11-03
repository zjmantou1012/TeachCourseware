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

![[类加载机制]]

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
[[Android Hook#Android Hook主要用到的技术]]

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

## 具体步骤
1. 实现插件管理类：
	1. 定义插件包的ClassLoader；
	2. 分别拿到插件包和宿主包中的dexElements数组并合并；
	3. 将新的dexElements数组更新到宿主的classloader中（dexElements最终存放 Dex 字节码数据的内存变量）。
2. 实现hook：
	1. **hook劫持AMS**，替换成宿主页面（在startActivity方法中将Intent信息替换成宿主页面）；
	2. **hook handler**，替换回插件页面（在ActivityThread中的handler，将ActivityClientRecord中的Intent信息替换回插件页面）；
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310241548763.png)

代码查看：[[Android 插件化#代码附件#hook&PluginManager]]

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
[AMS进阶——如何启动没有注册的Activity](https://blog.csdn.net/LVEfrist/article/details/123204061)
[Hook 插件化框架](https://blog.csdn.net/shulianghan/article/details/119521364)
[github链接]([https://github.com/han1202012/Plugin_Hook](https://github.com/han1202012/Plugin_Hook))


# 总结
## 占位 Activity
插件包中的 Activity 是通过正规流程 , 由 AMS 进行创建并加载的 , 但是 该 Activity 并没有在 AndroidManifest.xml 清单文件中注册 , 这里需要一个已经在清单文件注册的 Activity 欺骗系统 

## 插装式插件化 
是通过代理 Activity , 将 插件包加载的 字节码 Class 类 中 对应的 Activity 类作为一个普通的 Java 类 , 该普通的 Java 类有所有的 Activity 的业务逻辑 , 该 Activity 的生命周期 , 由代理 Activity 执行相关的生命周期方法

## hook 插件化
hook 插件化直接钩住系统中 Activity 启动流程的某个点



# 代码附件
## hook&PluginManager
```Java
package com.mantou.hookplugin;  
  
import android.content.Context;  
  
import java.lang.reflect.Array;  
import java.lang.reflect.Field;  
  
import dalvik.system.DexClassLoader;  
import dalvik.system.PathClassLoader;  
  
/**  
 * 使用 Hook 实现的插件使用入口  
 *  1. 加载插件包中的字节码  
 *  2. 直接通过 hook 技术, 钩住系统的 Activity 启动流程实现  
 *      ① Activity 对象创建之前 , 要做很多初始化操作 , 先在 ActivityRecord 中加载 Activity 信息  
 *          如果修改了该信息 , 将要跳转的 Activity 信息修改为插件包中的 Activity  
 *          原来的 Activity 只用于占位 , 用于欺骗 Android 系统  
 *      ② 使用 hook 技术 , 加载插件包 apk 中的 Activity  
 *      ③ 实现跳转的 Activity ( 插件包中的 )  
 *  3. 解决 Resources 资源冲突问题  
 *  ( 使用上述 hook 插件化 , 可以不用考虑 Activity 的声明周期问题 )  
 * *  插件包中的 Activity 是通过正规流程 , 由 AMS 进行创建并加载的  
 *      但是该 Activity 并没有在 AndroidManifest.xml 清单文件中注册  
 *      这里需要一个已经在清单文件注册的 Activity 欺骗系统  
 *  
 *  插装式插件化 是通过代理 Activity , 将插件包加载的字节码 Class 作为一个普通的 Java 类  
 *      该普通的 Java 类有所有的 Activity 的业务逻辑  
 *      该 Activity 的声明周期 , 由代理 Activity 执行相关的生命周期方法  
 *  hook 插件化 : hook 插件化直接钩住系统中 Activity 启动流程的某个点  
 *      使用插件包中的 Activity 替换占位的 Activity  
 */public class PluginManager {  
  
    /**  
     * 上下文  
     */  
    private Context mBase;  
  
    /**  
     * 单例  
     */  
    private static PluginManager instance;  
  
    public static PluginManager getInstance(Context context) {  
        if (instance == null) {  
            instance = new PluginManager(context);  
        }        return instance;  
    }  
    private PluginManager(Context context) {  
        this.mBase = context;  
    }  
    /**  
     * Application 启动后 , 调用该方法初始化插件化环境  
     *  加载插件包中的字节码  
     */  
    private void init() {  
        // 加载 apk 文件  
        loadApk();  
    }  
    private void loadApk() {  
        // 插件包的绝对路径 ,  /data/data/< package name >/files/        String apkPath = mBase.getFilesDir().getAbsolutePath() + "plugin.apk";  
        // 加载插件包后产生的缓存文件路径  
        // /data/data/< package name >/app_plugin_cache/  
        String cachePath =  
                mBase.getDir("plugin_cache", Context.MODE_PRIVATE).getAbsolutePath();  
        // 创建类加载器  
        DexClassLoader plugin_dexClassLoader =  
                new DexClassLoader(  
                        apkPath,                // 插件包路径  
                        cachePath,              // 插件包加载时产生的缓存路径  
                        null,   // 库的搜索路径, 可以设置为空  
                        mBase.getClassLoader()  // 父加载器, PathClassLoader  
                );  
  
        // 1. 反射 " 插件包 " 应用的 dexElement  
        // 执行步骤 :        // ① 反射获取 BaseDexClassLoader.class        // ② 反射获取 BaseDexClassLoader.calss 中的 private final DexPathList pathList 成员字段  
        // ③ 反射获取 plugin_dexClassLoader 类加载器中的 DexPathList pathList 成员对象  
        // ④ 反射获取 DexPathList.class        // ⑤ 反射获取 DexPathList.class 中的 private Element[] dexElements 成员变量的 Field 字段对象  
        // ⑥ 反射获取 DexPathList 对象中的 private Element[] dexElements 成员变量对象  
  
        // ① 反射获取 BaseDexClassLoader.class  
        // 通过反射获取插件包中的 dexElements        // 这种类加载是合并类加载 , 将所有的 Dex 文件 , 加入到应用的 dex 文件集合中  
        //  可参考 dex 加固 , 热修复 , 插装式插件化 的实现步骤  
        // 反射出 BaseDexClassLoader 类 , PathClassLoader 和 DexClassLoader        //  都是 BaseDexClassLoader 的子类  
        // 参考 https://www.androidos.net.cn/android/9.0.0_r8/xref/libcore/dalvik/src/main/java/dalvik/system/BaseDexClassLoader.java  
        Class<?> baseDexClassLoaderClass = null;  
        try {  
            baseDexClassLoaderClass = Class.forName("dalvik.system.BaseDexClassLoader");  
        } catch (ClassNotFoundException e) {  
            e.printStackTrace();  
        }  
        // ② 反射获取 BaseDexClassLoader.calss 中的 private final DexPathList pathList 成员字段  
        Field plugin_pathListField = null;  
        try {  
            plugin_pathListField = baseDexClassLoaderClass.getDeclaredField("pathList");  
            // 设置属性的可见性  
            plugin_pathListField.setAccessible(true);  
        } catch (NoSuchFieldException e) {  
            e.printStackTrace();  
        }  
        // ③ 反射获取 plugin_dexClassLoader 类加载器中的 DexPathList pathList 成员对象  
        // 根据 Field 字段获取 成员变量  
        //  DexClassLoader 继承了 BaseDexClassLoader, 因此其内部肯定有  
        //  private final DexPathList pathList 成员变量  
        Object plugin_pathListObject = null;  
        try {  
            plugin_pathListObject = plugin_pathListField.get(plugin_dexClassLoader);  
        } catch (IllegalAccessException e) {  
            e.printStackTrace();  
        }  
        // ④ 获取 DexPathList.class  
        // DexPathList 类中有 private Element[] dexElements 成员变量  
        // 通过反射获取该成员变量  
        // 参考 https://www.androidos.net.cn/android/9.0.0_r8/xref/libcore/dalvik/src/main/java/dalvik/system/DexPathList.java  
  
        // 获取 DexPathList pathList 成员变量的字节码类型 ( 也可以通过反射获得 )        // 获取的是 DexPathList.class        Class<?> plugin_dexPathListClass = plugin_pathListObject.getClass();  
  
        // ⑤ 反射获取 DexPathList.class 中的 private Element[] dexElements 成员变量的 Field 字段对象  
        Field plugin_dexElementsField = null;  
        try {  
            plugin_dexElementsField = plugin_dexPathListClass.getDeclaredField("dexElements");  
            // 设置属性的可见性  
            plugin_dexElementsField.setAccessible(true);  
        } catch (NoSuchFieldException e) {  
            e.printStackTrace();  
        }  
        // ⑥ 反射获取 DexPathList 对象中的 private Element[] dexElements 成员变量对象  
        Object plugin_dexElementsObject = null;  
        try {  
            plugin_dexElementsObject = plugin_dexElementsField.get(plugin_pathListObject);  
        } catch (IllegalAccessException e) {  
            e.printStackTrace();  
        }  
  
  
        // 2. 反射 " 宿主 " 应用的 dexElement        // 执行步骤 :        // ① 反射获取 BaseDexClassLoader.class        // ② 反射获取 BaseDexClassLoader.calss 中的 private final DexPathList pathList 成员字段  
        // ③ 反射获取 PathClassLoader 类加载器中的 DexPathList pathList 成员对象  
        // ④ 反射获取 DexPathList.class        // ⑤ 反射获取 DexPathList.class 中的 private Element[] dexElements 成员变量的 Field 字段对象  
        // ⑥ 反射获取 DexPathList 对象中的 private Element[] dexElements 成员变量对象  
  
        // ① 反射获取 BaseDexClassLoader.class        Class<?> host_baseDexClassLoaderClass = null;  
        try {  
            host_baseDexClassLoaderClass = Class.forName("dalvik.system.BaseDexClassLoader");  
        } catch (ClassNotFoundException e) {  
            e.printStackTrace();  
        }  
        // ② 反射获取 BaseDexClassLoader.calss 中的 private final DexPathList pathList 成员字段  
        Field host_pathListField = null;  
        try {  
            host_pathListField = host_baseDexClassLoaderClass.getDeclaredField("pathList");  
            // 设置属性的可见性  
            host_pathListField.setAccessible(true);  
        } catch (NoSuchFieldException e) {  
            e.printStackTrace();  
        }  
        // ③ 反射获取 DexClassLoader 类加载器中的 DexPathList pathList 成员对象  
        // 根据 Field 字段获取 成员变量  
        //  DexClassLoader 继承了 BaseDexClassLoader, 因此其内部肯定有  
        //  private final DexPathList pathList 成员变量  
        PathClassLoader host_pathClassLoader = (PathClassLoader) mBase.getClassLoader();  
        Object host_pathListObject = null;  
        try {  
            host_pathListObject = host_pathListField.get(host_pathClassLoader);  
        } catch (IllegalAccessException e) {  
            e.printStackTrace();  
        }  
        // ④ 获取 DexPathList.class  
        // DexPathList 类中有 private Element[] dexElements 成员变量  
        // 通过反射获取该成员变量  
        // 参考 https://www.androidos.net.cn/android/9.0.0_r8/xref/libcore/dalvik/src/main/java/dalvik/system/DexPathList.java  
  
        // 获取 DexPathList pathList 成员变量的字节码类型 ( 也可以通过反射获得 )        // 获取的是 DexPathList.class        Class<?> host_dexPathListClass = host_pathListObject.getClass();  
  
        // ⑤ 反射获取 DexPathList.class 中的 private Element[] dexElements 成员变量的 Field 字段对象  
        Field host_dexElementsField = null;  
        try {  
            host_dexElementsField = host_dexPathListClass.getDeclaredField("dexElements");  
            // 设置属性的可见性  
            host_dexElementsField.setAccessible(true);  
        } catch (NoSuchFieldException e) {  
            e.printStackTrace();  
        }  
        // ⑥ 反射获取 DexPathList 对象中的 private Element[] dexElements 成员变量对象  
        Object host_dexElementsObject = null;  
        try {  
            host_dexElementsObject = host_dexElementsField.get(host_pathListObject);  
        } catch (IllegalAccessException e) {  
            e.printStackTrace();  
        }  
  
        // 3. 合并 “插件包“ 与 “宿主“ 中的 Element[] dexElements  
        // 将两个 Element[] dexElements 数组合并 ,        //  合并完成后 , 设置到 PathClassLoader 中的  
        //  DexPathList pathList 成员的 Element[] dexElements 成员中  
  
        // 获取 “宿主“ 中的 Element[] dexElements 数组长度  
        int host_dexElementsLength = Array.getLength(host_dexElementsObject);  
        // 获取 “插件包“ 中的 Element[] dexElements 数组长度  
        int plugin_dexElementsLength = Array.getLength(plugin_dexElementsObject);  
  
        // 获取 Element[] dexElements 数组中的 , 数组元素的 Element 类型  
        // 获取的是 Element.class        Class<?> elementClazz = host_dexElementsObject.getClass().getComponentType();  
  
        // 合并后的 Element[] dexElements 数组长度  
        int new_dexElementsLength = plugin_dexElementsLength + host_dexElementsLength;  
  
        // 创建 Element[] 数组 , elementClazz 是 Element.class 数组元素类型  
        Object newElementsArray = Array.newInstance(elementClazz, new_dexElementsLength);  
  
        //  为新的 Element[] newElementsArray 数组赋值  
        //      先将 “插件包“ 中的 Element[] dexElements 数组放入到新数组中  
        //      然后将 “宿主“ 中的 Element[] dexElements 数组放入到新数组中  
        for (int i = 0; i < new_dexElementsLength; i++) {  
            if (i < plugin_dexElementsLength) {  
                // “插件包“ 中的 Element[] dexElements 数组放入到新数组中  
                Array.set(newElementsArray, i, Array.get(plugin_dexElementsObject, i));  
            } else {  
                //  “宿主“ 中的 Element[] dexElements 数组放入到新数组中  
                Array.set(newElementsArray, i, Array.get(host_dexElementsObject, i - plugin_dexElementsLength));  
            }        }  
  
  
        // 4. 重新设置 PathClassLoader 中的 DexPathList pathList 成员的 Element[] dexElements 属性值  
        Field elementsFiled = null;  
        try {  
            elementsFiled = host_pathListObject.getClass().getDeclaredField("dexElements");  
        } catch (NoSuchFieldException e) {  
            e.printStackTrace();  
        }        elementsFiled.setAccessible(true);  
  
        //  设置 DexPathList pathList 的 Element[] dexElements 属性值  
        //  host_pathListObject 是原来的属性值  
        //  newElementsArray 是新的合并后的 Element[] dexElements 数组  
        //  注意 : 这里也可以使用 host_dexElementsField 字段进行设置  
        try {  
            elementsFiled.set(host_pathListObject, newElementsArray);  
        } catch (IllegalAccessException e) {  
            e.printStackTrace();  
        }  
  
    }  
}
```