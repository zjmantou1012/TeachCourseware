---
author: zjmantou
title: 懂了！时间复杂度O(1)，O(logn) ，O(n)，O(nlogn)..._数据结构_Ayue、_InfoQ写作社区
time: 2024-03-28 周四
tags:
  - 技术
  - 资料
  - 算法
---



## 写在前面

在学习数据结构和算法的时候，经常会碰到 O(1),O(n)等等用来表示时间和空间复杂度，那这到底是什么意思。我们对于同一个问题经常有不同的解决方式，比如排序算法就有十种经典排序（快排，归并排序等），虽然对于排序的结果相同，但是在排序过程中消耗时间和资源却是不同。

对于不同排序算法之间的衡量方式就是通过程序执行所占用的**时间**和**空间**两个维度去考量。

## 高中数学

### 函数

设 _A_、_B_ 是非空的数集，如果按照某个确定的对应关系 _f_，使对于集合 _A_ 中的任意一个数 _x_，在集合 _B_ 中都有唯一确定的数 _f_(_x_)和它对应，那么就称 _f_：_A_→_B_ 为从集合 _A_ 到集合 _B_ 的一个函数。记作：_y_=_f_(_x_)，_x_∈_A_。其中，_x_ 叫做自变量，_x_ 的取值范围 _A_ 叫做函数的定义域；与 _x_ 的值相对应的 _y_ 值叫做函数值，函数值的集合{_f_(_x_)| _x_∈_A_ }叫做函数的值域。

例：已知 _f_(_x_)的定义域为[3,5],求*f(2x-1)*的定义域。

![](file:///Users/mantou/.config/joplin-desktop/resources/262dc8a56c0a44c4a805eb29573965aa.png)

### 幂函数

### 指数函数

![](file:///Users/mantou/.config/joplin-desktop/resources/38946a92499c47ddb18bec333eda30d4.png)

### 对数函数

![](file:///Users/mantou/.config/joplin-desktop/resources/75d9cec15bdc4f1cba2ae7ffa5322982.png)

## 时间复杂度

若存在函数 f(n)，使得当 n 趋近于无穷大时，T(n)/ f(n)）的极限值为不等于零的常数，则称 f(n)是 T(n)的同数量级函数。记作 T(n)= O(f(n))，称 O(f(n))为算法的渐进时间复杂度，简称时间复杂度。

**简单理解就是一个算法或是一个程序在运行时，所消耗的时间（或者代码被执行的总次数）。**

在下面的程序中：

```java
int sum(int n) {
①   int value = 0;
②   int i = 1;
③   while (i <= n) {
④       value = value + i;
⑤       i++;
    }
⑥   return value;
}

假设n=100，该方法的执行次数为①(1次)、②（1次）、③（100次）、④（100次）、⑤（100次）、⑥（1次）
合计1+1+100+100+100+1 = 303次
```

上面的结果如果用函数来表示为：_f(n)_ = 3n+3，那么在计算机算法中的表示方法如下。

### 表示方法

大 O 表示法：算法的时间复杂度通常用大 O 来表示，定义为 **T(n) = O(f(n))**，其中 T 表示时间。

即：T(n) = O(3n+3)

这里有个重要的点就是时间复杂度关心的是**数量级**，其原则是：

1. 省略常数，如果运行时间是常数量级，用常数 1 表示
    
2. 保留最高阶的项
    
3. 变最高阶项的系数为 1
    

如 2n <sup>3</sup> + 3n<sup>2</sup> + 7，省略常数变为 O(2n <sup>3</sup> + 3n<sup>2</sup>)，保留最高阶的项为 O(2n <sup>3</sup> )，变最高阶项的系数为 1 后变为 O(n <sup>3</sup> )，即 O(n <sup>3</sup> )为 2n <sup>3</sup> + 3n<sup>2</sup> + 7 的时间复杂度。

同理，在上面的程序中 T(n) = O(3n+3)，其时间复杂度为 O(n)。

注：只看最高复杂度的运算，也就是上面程序中的内层循环。

### 时间复杂度的阶

时间复杂度的阶主要分为以下几种

#### 常数阶 O(1)

```Java
int n = 100;
System.out.println("常数阶：" + n);
```


不管 n 等于多少，程序始终只会执行一次，即 T(n) = O(1)

![](file:///Users/mantou/.config/joplin-desktop/resources/a7835dbf0d6745ddbe7f45fc4acb2afc.png)

#### 对数阶 O(log<sup>n</sup>)

```Java
// n = 32 则 i=1,2,4,8,16,32
for (int i = 1; i <= n; i = i * 2) {
    System.out.println("对数阶：" + n);
}
```


i 的值随着 n 成对数增长，读作 2 为底 n 的对数，即 _f_(_x_) = log<sub>2</sub><sup>n</sup>，T(n) = O( log<sub>2</sub><sup>n</sup>)，简写为 O(log<sup>n</sup>)

则对数底数大于 1 的象限通用表示为：

![](file:///Users/mantou/.config/joplin-desktop/resources/d5d615a4f44849ac94225f8b4803374d.png)

#### 线性阶 O(n)

```Java
for (int i = 1; i < n; i++) {
    System.out.println("线性阶：" + n);
}
```


n 的值为多少，程序就运行多少次，类似函数 y = f(x)，即 T(n) = O(n)

![](file:///Users/mantou/.config/joplin-desktop/resources/055c82cc17ae4c53a065bf02a67fb35b.png)

#### 线性对数阶 O(nlog<sup>n</sup>)

```Java
for (int m = 1; m <= n; m++) {
    int i = 1;
    while (i < n) {
        i = i * 2;
        System.out.println("线性对数阶：" + i);
    }
}
```


线性对数阶 O(nlog<sup>n</sup>)其实非常容易理解，将对数阶 O(log<sup>n</sup>)的代码循环 n 遍的话，那么它的时间复杂度就是 n * O(log<sup>n</sup>)，也就是了 O(nlog<sup>n</sup>)，归并排序的复杂度就是 O(nlog<sup>n</sup>)。

若 n = 2 则程序执行 2 次，若 n=4,则程序执行 8 次，依次类推

![](file:///Users/mantou/.config/joplin-desktop/resources/5e1c3e540ca449579dd18490c24764fe.png)

#### 平方阶 O(n<sup>2</sup>)

```Java
for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= n; j++) {
        System.out.println("平方阶：" + n);
    }
}
```


若 n = 2，则打印 4 次，若 n = 3，则打印 9，即 T(n) = O(n<sup>2</sup>)

![](file:///Users/mantou/.config/joplin-desktop/resources/4daefc68d889496d8875fac8abab35cf.png)

同理，立方阶就为 O(n<sup>3</sup>)，如果 3 改为 k，那就是 k 次方阶 O(n<sup>k</sup>)，相对而言就更复杂了。

以上 5 种时间复杂度关系为：

![](file:///Users/mantou/.config/joplin-desktop/resources/d303decafdd04b0981315c2cc5b7aa07.png)

从上图可以得出结论，当 x 轴 n 的值越来越大时，y 轴耗时的时长为：

O(1) < O(log<sup>n</sup>) < O(n) < O(nlog<sup>n</sup>) < O(n<sup>2</sup>)

在编程算法中远远不止上面 4 种，比如 O(n<sup>3</sup>)，O(2<sup>n</sup>)，O(n!)，O(n<sup>k</sup>)等等。

这些是怎么在数学的角度去证明的，感兴趣的可以去看看[主定理](https://xie.infoq.cn/link?target=http%3A%2F%2Fzh.m.wiki.sxisa.org%2Fwiki%2F%25E4%25B8%25BB%25E5%25AE%259A%25E7%2590%2586 "https://xie.infoq.cn/link?target=http%3A%2F%2Fzh.m.wiki.sxisa.org%2Fwiki%2F%25E4%25B8%25BB%25E5%25AE%259A%25E7%2590%2586")。

注：以下数据来自于[Big-O Cheat Sheet](https://xie.infoq.cn/link?target=https%3A%2F%2Fwww.bigocheatsheet.com%2F "https://xie.infoq.cn/link?target=https%3A%2F%2Fwww.bigocheatsheet.com%2F")，常用的**大 O 标记法**列表以及它们与不同大小输入数据的性能比较。

### 常见数据结构操作的复杂度

### 数组排序算法的复杂度

## 空间复杂度

**空间复杂度表示的是算法的存储空间和数据之间的关系，即一个算法在运行时，所消耗的空间。**

### 空间复杂度的阶

空间复杂度相对于时间复杂度要简单很多，我们只需要掌握常见的 O(1)，O(n)，O(n<sup>2</sup>)。

#### 常数阶 O(1)

#### 线性阶 O(n)

#### 平方阶 O(n<sup>2</sup>)

## 总结

写这篇文章的目的在于，我在更新[栈和队列](https://xie.infoq.cn/link?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F6983959426243756039 "https://xie.infoq.cn/link?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F6983959426243756039")的时候，留下了一个问题**栈的入栈和出栈操作与队列的插入和移除的时间复杂度是否相同**，确实是相同的 ，都用了 O(1)。由此我想到，对于刚开始或者说是不太了解复杂度的同学，碰到此类的问题，或者是我在跟后续数据结构和算法文章的时候不懂，十分的不友好，于是就写了一篇关于复杂 度的文章。

本文只对时间复杂度和空间复杂度做了简单介绍，有错误可以指正，不要硬杠，杠就是我输。

![](file:///Users/mantou/.config/joplin-desktop/resources/2a36b16f493042c38d6142f0270dc67a.png)

## 参考

- [一套图搞懂时间复杂度](https://xie.infoq.cn/link?target=https%3A%2F%2Fblog.csdn.net%2Fqq_41523096%2Farticle%2Fdetails%2F82142747 "https://xie.infoq.cn/link?target=https%3A%2F%2Fblog.csdn.net%2Fqq_41523096%2Farticle%2Fdetails%2F82142747")
    
- [算法的时间复杂度和空间复杂度](https://xie.infoq.cn/link?target=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2FBV1Cv41147sJ%3Ft%3D49 "https://xie.infoq.cn/link?target=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2FBV1Cv41147sJ%3Ft%3D49")
    

**都是自己人，点赞关注我就收下了🤞**

# [原文链接](https://xie.infoq.cn/article/4f20bb83e3693afc9746d35de)

