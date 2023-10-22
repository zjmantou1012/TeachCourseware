---
author: zjmantou
title: Android Hook
time: 2023-10-22 周日
tags:
  - Android
  - Hook
  - 技术
---
Hook技术又称为钩子技术，常用于系统入侵。

面向切面工程思想。

Android中的Hook技术：
通过分析 Android 系统源码执行 , 通过 ***动态注入技术*** , 在代码运行的某个阶段 , 注入开发者自定义的代码。  

Hook技术点：
阅读源码，找到可替换的hook点，一般找静态类、变量和方法；

# 常用的动态注入技术
1. **编译时修改字节码数据**：代码编译时修改 Class 字节码数据 , 如 Dagger；
2. **运行时修改字节码数据**：运行时可以修改字节码文件数据 , 达到代码入侵的效果。

# Android Hook主要用到的技术
1. 反射机制；
2. 代理机制：动态代理，静态代理；

## 动态代理代码示例

```Java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;

public class Main {

    public static void main(String[] args) {
        // 1. 创建目标对象
        Subject subject = new Subject();

        // 2. 获取目标对象类加载器
        ClassLoader classLoader = subject.getClass().getClassLoader();

        // 3. 获取接口 Class 数组, Subject 只实现了一个 AInterface 接口
        Class<?>[] interfaces = subject.getClass().getInterfaces();

        // 4. 创建 InvocationHandler , 传入被代理的目标对象 , 处理该目标对象中被代理的函数
        InvocationHandler invocationHandler = new AInvocationHandler(subject);

        // 5. 动态代理 :
        //    ① jdk 根据传入的参数信息 , 在内存中动态的创建 与 .class 字节码文件等同的字节码数据
        //    ② 将字节码数据 转为 对应的 字节码类型
        //    ③ 使用反射调用 newInstance 创建动态代理实例
        AInterface proxy = (AInterface) Proxy.newProxyInstance(
                classLoader,
                interfaces,
                invocationHandler);
        // 正式调用被动态代理的类
        proxy.request();
    }
}

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class AInvocationHandler implements InvocationHandler {

    /**
     * 被代理的对象
     */
    Object target;

    public AInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        before();
        Object object = method.invoke(target, args);
        after();
        return object;
    }

    /**
     * 被代理对象方法调用之前执行
     */
    private void before(){
        System.out.println("AInvocationHandler before");
    }

    /**
     * 被代理对象方法调用之后执行
     */
    private void after(){
        System.out.println("AInvocationHandler after");
    }
}

```

#### 参考链接
[Android Hook 技术深入解析以及简单实战](https://juejin.cn/post/6844903806900109319?searchId=20230915190157C648EF222B2846EFDAFB)
[盘点Android常用Hook技术](https://zhuanlan.zhihu.com/p/109157321)