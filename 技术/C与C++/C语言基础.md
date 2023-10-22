---
author: zjmantou
title: C语言基础
time: 2023-10-10 周二
tags:
  - C
---
# stdio.h与stdlib.h
## stdio.h

standard input&output缩写，有关标准输入/输出信息；
## stdlib.h

- 类型：如size_t、wchar_t、div_t、ldiv_t和lldiv_t；
- 宏：如EXIT_FAILURE、EXIT_SUCCESS、RAND_MAX和MB_CUR_MAX等；
- 常用等函数：如malloc()，calloc()、free()、system()等。

# 指针、指针变量

指针是指指向内存空间的地址；  
可以直接访问硬件（如OpenGL显卡绘制），可以快速传递数据，可以返回一个以上的值（如返回一个数组或者结构体的指针）；可以表示复杂的数据结构，可以方便的处理字符串，可以有助于理解面向对象。  
指针变量：如int* p，p就是一个指针变量；

# 操作系统内存
- .data区：即常量区，Java中的常量，C中的宏；
- .code区：存放函数；
- 栈空间：一块连续的空间，效率较堆空间高；
- 堆空间：不连续空间，有碎片。  

当执行main中调用子函数时，先会在.code区中寻找该函数，然后为子函数中的变量在栈内存中分配内存空间，执行完子函数后，子函数中变量（局部变量出栈，成员变量保留）会执行出栈，子函数中的变量所对应的内存空间会被销毁，且这块内存空间不能被其他变量所使用。  

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310101733712.png)


# 数组

不会有数组越界问题，当数组中的值为被赋值时，默认是0。  
数组的名字与数组第一个元素的地址相同。