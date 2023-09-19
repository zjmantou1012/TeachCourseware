---
author: zjmantou
title: （2）JNI开发必学C-C++使用实践
time: 2023-09-19 周二
tags:
  - Android
  - NDK
  - JNI
---
**JNI开发系列目录**

1. [JNI开发必学C++基础](https://blog.csdn.net/luo_boke/article/details/126908373 "https://blog.csdn.net/luo_boke/article/details/126908373")
2. [JNI开发必学C++使用实践](https://blog.csdn.net/luo_boke/article/details/126920916 "https://blog.csdn.net/luo_boke/article/details/126920916")
3. [Android Studio 4.0.+NDK项目开发详细教学](https://blog.csdn.net/luo_boke/article/details/107306531 "https://blog.csdn.net/luo_boke/article/details/107306531")
4. [Android NDK与JNI的区别有何不同？](https://blog.csdn.net/luo_boke/article/details/107234358 "https://blog.csdn.net/luo_boke/article/details/107234358")
5. [Android Studio 4.0.+NDK .so库生成打包](https://blog.csdn.net/luo_boke/article/details/109362013 "https://blog.csdn.net/luo_boke/article/details/109362013")
6. [Android JNI的深度进阶学习](https://blog.csdn.net/luo_boke/article/details/109455910 "https://blog.csdn.net/luo_boke/article/details/109455910")
7. [Android Studio 4.0.+NDK开发 This files is not part of the project](https://blog.csdn.net/luo_boke/article/details/109318721 "https://blog.csdn.net/luo_boke/article/details/109318721")

---

**`以Android studio 7.0.0来分析讲解，所以是Android最新版NDK项目创建，其截图可能与低版本不一样。`**

---

## 前言

上一篇博文[JNI开发必学C++基础](https://blog.csdn.net/luo_boke/article/details/126908373 "https://blog.csdn.net/luo_boke/article/details/126908373")介紹了C/C++的相关基础知识，本篇博文来讲解C/C++的相关语法实操。

C++学习网站：[https://en.cppreference.com/w/](https://en.cppreference.com/w/ "https://en.cppreference.com/w/")

## 基础语法

约定俗成的操作是 .h 文件主要负责类成员变量和方法的声明； .cpp 文件主要负责成员变量和方法的定义。  
1 . 属性与函数声明

```
//A.h文件
class A
{
private: //私有属性声明
    int a; 
    void add();

protected: //子类可见声明
    int b;
    void sum(int i);

public: //公开属性声明
    int c = 2;
    int count(int j);
    A(int a, int b); // 构造函数
    ～A(); //析构函数
};
```

**构造函数**在类创建的时候调用，**析构函数**在类被删除的时候调用，主要用于释放内部变量和内存。

2.函数实现

```
//A.cpp文件
//实现构造方法
A::A(int a1, int b1) {
    a = a1;
    b = b1;
}

//实现析构函数
A::~A() {
    
}

//实现普通函数
void sum(int i){
    b+=i
}
```

3. 函数调用

```
#include<iostream>
#include "A.h"
using namespace std;
int main() {
    // 手动分配内存 （不推荐使用）
    A *a = new A(1,2);
    a->sum(3);
    delete a;

    // 自动分配内存 （推荐使用）
    A a2 = A(1,2);
    a2.sum(4);
    return 0;
}
```

---

## 指针

**`指针`**：是一个变量，这个变量的值是另一个变量的内存地址。也就是说，指针是一个指向内存地址的变量。  
Java 中没有指针的概念的，但是其实 Java 中除了基本数据类，大部分情况下使用都是指针。

```
int main ()
{
   int  var = 20;   // 实际变量的声明
   int  *ip;        // 指针变量的声明
   ip = &var;       // 在指针变量中存储 var 的地址
   cout << "Value of var variable: ";
   cout << var << endl;
   // 输出在指针变量中存储的地址
   cout << "Address stored in ip variable: ";
   cout << ip << endl;
   // 访问指针中地址的值
   cout << "Value of *ip variable: ";
   cout << *ip << endl;
   return 0;
}

输出结果
Value of var variable: 20
Address stored in ip variable: 0xbfc601ac
Value of *ip variable: 20
```

`*` ：有两个作用：  
i. 用于定义一个指针： type *var_name; ，var_name 是一个指针变量，如 int *p;  
ii. 用于对一个指针取内容： *var_name， 如 *p 的值是 1。

**`&`** ：是一个取址符号  
其用于获取一个变量所在的内存地址。如 &a; 的值是 a 所在内存的位置，即 a 的地址。

---

**指针使用案例**

指针的使用举例，便于大家理解指针。

```
int main() {
    
    //-----1-------
    A a = A(); // 定变量 a
    a.i = 1;   // 修改 a 中的变量
    A b = a;   // 定义变量 b ，赋值为 a
    A *c = &a; // 定义指针 c，指向 a
    printf("%d, %d, %d\n", a.i, b.i, c->i);
    // 输出：1, 1, 1
    
    b.i = 2; //修改 b 中的变量
    printf("%d, %d, %d\n", a.i, b.i, c->i);
    // 输出：1, 2, 1
    
    c->i = 3; //修改 c 中的变量
    printf("%d, %d, %d\n", a.i, b.i, c->i);
    // 输出：3, 2, 3
    
    // 打印地址
    printf("%d, %d, %d\n", &a, &b, c);
    // 输出：-1861360224, -1861360208, -1861360224
    return 0;
}
```

1. 通过 **普通变量** 赋值的时候，系统创建了一个新的独立的内存块，如 b，对 b 的修改，只影响其本身；
2. 通过 **指针变量** 赋值时，系统没有创建新的内存块，而是将指针指向了已存在的内存块，如 c ， 任何对 c 的修改，都将影响原来的变量，如 a。

**new与delete**  
指针变量可以通过new动态创建。

```
A *a = new A(1,2)// 无new，main 函数结束后，系统会自动回收内存
    A *b = new A(1,2);// new 方式创建指针，系统不会自动回收内存，要手动 delete
    b->sum(3); 
    delete b;// 手动删除，回收内存
```

**`注意`**：通过 `new` 创建的指针需要我们自己手动回收内存，否则将会导致内存泄漏。回收内存则是通过`delete`关键字进行的。

---

## 引用

引用与指针的区别

1. 不存在空引用。引用必须连接到一块合法的内存。
2. 一旦引用被初始化为一个对象，就不能被指向到另一个对象。指针可以在任何时候指向到另一个对象。
3. 引用必须在创建时被初始化。指针可以在任何时间被初始化。

## 纯虚函数

在 Java 中，我们经常会使用 interface 或 abstract 来定义一些接口，方便代码规范和拓展，但是在 C++ 没有这样的方法，但是可以有类似的实现，那就是：[纯虚函数](https://so.csdn.net/so/search?q=%E7%BA%AF%E8%99%9A%E5%87%BD%E6%95%B0&spm=1001.2101.3001.7020 "https://so.csdn.net/so/search?q=%E7%BA%AF%E8%99%9A%E5%87%BD%E6%95%B0&spm=1001.2101.3001.7020")。

```
class A {
public:
    // 声明一个纯虚函数
    virtual void f() = 0;
}

class B : public A {
public: 
    // 子类必须实现 f ，否则编译不通过
    void f() {
        printf("b\n");
    };
};
```

有虚函数的类如A是一个抽象类，不能被直接定义使用。

## 容器

|容器分类|类名|概要|
|---|---|---|
|顺序容器|array|不支持动态扩容，采用数组的数据结构进行数据存储|
|顺序容器|vector|支持动态扩容，采用数组的数据结构进行数据存储|
|顺序容器|list|支持动态扩容，采用链表的数据结构进行数据存储|
|关联容器|set|默认按照value进行升序排序,值唯一。二叉树数据结构|
|关联容器|map|默认按照key进行升序排序,key唯一。二叉树数据结构|
|关联容器|multiset|默认按照value进行升序排序,值不唯一。二叉树数据结构|
|关联容器|multimap|默认按照key进行升序排序,key不唯一。二叉树数据结构|
|无序关联容器|unordered_set|对应容器的无序版本。哈希表数据结构|
|无序关联容器|unordered_map|对应容器的无序版本。哈希表数据结构|
|无序关联容器|unordered_multiset|对应容器的无序版本。哈希表数据结构|
|无序关联容器||unordered_multimap|

---

## 总结

C++与 Java 相似，又存在差异的一些基础知识，由于面向对象语言都存在一定的相似性，相信有了以上的基础之后，你就可以比较通畅地阅读一些 C++ 代码了。

---

**相关链接**：

1. [JNI开发必学C++基础](https://blog.csdn.net/luo_boke/article/details/126908373 "https://blog.csdn.net/luo_boke/article/details/126908373")
2. [JNI开发必学C++使用实践](https://blog.csdn.net/luo_boke/article/details/126920916 "https://blog.csdn.net/luo_boke/article/details/126920916")
3. [Android Studio 4.0.+NDK项目开发详细教学](https://blog.csdn.net/luo_boke/article/details/107306531 "https://blog.csdn.net/luo_boke/article/details/107306531")
4. [Android NDK与JNI的区别有何不同？](https://blog.csdn.net/luo_boke/article/details/107234358 "https://blog.csdn.net/luo_boke/article/details/107234358")
5. [Android Studio 4.0.+NDK .so库生成打包](https://blog.csdn.net/luo_boke/article/details/109362013 "https://blog.csdn.net/luo_boke/article/details/109362013")
6. [Android JNI的深度进阶学习](https://blog.csdn.net/luo_boke/article/details/109455910 "https://blog.csdn.net/luo_boke/article/details/109455910")
7. [Android Studio 4.0.+NDK开发 This files is not part of the project](https://blog.csdn.net/luo_boke/article/details/109318721 "https://blog.csdn.net/luo_boke/article/details/109318721")

**博客书写不易，您的点赞收藏是我前进的动力，千万别忘记点赞、 收藏 ^ _ ^ !**