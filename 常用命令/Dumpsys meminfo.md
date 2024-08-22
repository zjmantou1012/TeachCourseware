---
author: zjmantou
title: Dumpsys meminfo
time: 2024-08-22 周四
tags:
  - 笔记
  - shell
---
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202408221724777.png)

| Item | 全称                    | 含义   | 等价                  |
| ---- | --------------------- | ---- | ------------------- |
| USS  | Unique Set Size       | 物理内存 | 进程独占的内存             |
| PSS  | Proportional Set Size | 物理内存 | Pss =Uss+按比例包含共享库   |
| RSS  | Resient Set Size      | 物理内存 | RSS=USS+包含共享库       |
| VSS  | Virtual Set Size      | 虚拟内存 | VSS = RSS+未分配实际物理内存 |


虚拟内存区域（VMA）的几种状态：
- Resident：该页被映射到物理内存页。
	- Clean：页面的内容与磁盘上的内容相同，当内存不足时可以清除Clean pages，当再次需要时，内核可以从底层文件读取它们来重新创建其内容。
	- Dirty：页面的内容与磁盘不同，或者为匿名页。无法清除，因为这样会导致数据丢失。但可以在磁盘或者ZRAM上进行交换。
- Swapped：脏页可以写入磁盘上的交换文件（Linux桌面版）或压缩compressed（Android和CrOS上通过ZRAM）。该页面保持交换状态，直到其虚拟地址上发生新的页面错误，此时内核会将其带回主内存中。 
- Not present：页面上没有发生页错误，或者页面时Clean的，稍后被驱逐。 

一般要减少脏内存，因为其不能被回收；

## 横坐标

- **Pss Total**：Pss是指实际使用的物理内存，考虑了在进程之间共享RAM页的情况。进程独占的RAM页会直接计入其PSS值，而与其他进程共享的RAM页则会按相应比例计入PSS值。例如，两个进程之间共享的RAM页会将其一半的大小 分别计入这两个进程的PSS中，Pss Total 就是指某一项Pss的总值。
- **Private Clean**：干净内存；进程私有的，相对磁盘数据没有使用修改的内存。
- **Private Dirty**：脏内存；进程私有，是仅分配给应用堆的实际RAM，包含了您自己的分配和zygote分配页，这些分配页自从zygote派生您的应用进程以来已被修改。
- **SwapPss Dirty**：一些Android设备确实使用了内存交换，但它们使用的是内存而不是闪存。Linux有一个称为ZRAM的特性，它可以压缩页面，然后将它们交换到一个特殊的RAM区域，并在需要时再次解压它们。因此，“交换肮脏”中列出的页面很可能是在ZRAM中。

## 纵坐标 

1. Native Heap	在Native Code中使用malloc分配出的内存
2.	Dalvik Heap	Dalvik 虚拟机（Java 代码）分配的空间，不包括它自身的开销，Dalvik堆中和zygote进程共享的部分算是sharedDirty
3.	Dalvik Other	类数据结构和索引占据的内存
3.	Stack	堆内存
4.	Ashmem	匿名共享内存，此类内存与cache shrinker关联，可以控制cache shrinker在适当时机回收这些共享内存
5.	Other dev	内存drvier占用的内存
6.	.so mmap	映射的.so(native) 占用的内存
7.	.jar mmap	Java文件代码占用内存
8.	.apk mmap	apk代码占用内存
8.	.dex mmap	映射的.dex(Dalvik 或ART)代码占用的内存
9.	.oat mmap	代码映射占用的RAM量。此映像在所有应用之间共享，不受特定应用影响
10. .art mmap	堆映像占用的RAM量，此映像在所有应用之间共享，不受特定应用影响。尽管ART映射包含Object实例，它仍然不会计入您的堆大小
11. other mmap	其他文件占用的内存