---
author: zjmantou
title: CPrimerPlus（第6版）
time: 2023-10-17 周二
tags:
  - C
  - cpp
---

# C语言基础
- 开发环境：VSCode
- 编译运行：
```clang
cc xxx.c
./a.out
```
## 调用两个函数
```C
#include <stdio.h>
void butler(void); /* ANSI/ISO C函数原型 void代表空 */

int main(void)
{
printf("I will summon the butler funciont.\n");
butler();
printf("Yes.Bring me some tea and writeable DVDs.\n");
return 0;
}

void butler(void){
printf("Young rang,sir?\n");
}
```
