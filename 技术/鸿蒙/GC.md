---
author: zjmantou
title: GC
time: 2025-02-18 周二
tags:
  - 鸿蒙
  - 技术
  - 垃圾收集器
---

# GC类型

## 引用计数

- 优点：算法简单，回收及时
- 内存和赋值额外开销，循环引用问题

## 根对象追踪

- 优点：解决了循环引用问题，对内存和赋值没有额外开销
- 缺点：算法较为复杂，有短暂的STW阶段，会有延迟，导致较多的浮动垃圾

### 三种类型

#### 标记-清除

![标记-清除算法](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250217164522.68426065344233712887499018880185:50001231000000:2800:012024E545A37453943164ED6873CAE12899A76B31DB17CF885563940856FE1A.png?needInitFileName=true?needInitFileName=true)

回收效率高，但会导致内存碎片化，降低内存分配效率。 

#### 标记-复制

![标记-复制算法](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250217164522.28989730488414088682224917886408:50001231000000:2800:9399BA687CB1AF034C13655BAA454C0C8D7D0E79F87248E8E23517D09FD3FF15.png?needInitFileName=true?needInitFileName=true) 

解决了内存碎片问题，一次遍历，效率高，但空间利用率低。

#### 标记-整理 

![标记-整理算法](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250217164522.10929756944893721639680052853298:50001231000000:2800:C0A7926022ECB1A05A5E1CF8DE979BDDD500909C456410A4C611E12DB8DFEC65.png?needInitFileName=true?needInitFileName=true) 

既解决了空间碎片问题，又提高利用率，但性能开销稍大。 

# HPP GC

HPP GC（High Performance Partial Garbage Collection）,即高性能部分垃圾回收：
- 分代模型、
- 混合算法
- GC流程优化

![分代模型](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250217164523.14739399340380886752961710274385:50001231000000:2800:D54083A13CB7CC3DBF21DB0E9751FA82374EC4F84B384295B937D367A8791504.png?needInitFileName=true?needInitFileName=true) 

## 流程优化 

HPP GC流程中引入了大量的并发和并行优化，以减少对应用性能的影响。采用了并发+并行标记（Marking）、并发+并行清扫（Sweep）、并行复制/整理（Evacuation）、并行回改（Update）和并发清理（Clear）执行GC任务。 

# Heap结构及其配置参数 

![Heap结构](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250217164523.06568311506997438106067628919510:50001231000000:2800:734EFEACA1E5884904136E18E4B75C4CA70BF1264D089574E41EFE7F1DC06E6F.png?needInitFileName=true?needInitFileName=true) 

- SemiSpace：年轻代（Young Generation），存放新创建出来的对象，存活率低，主要使用copying算法进行内存回收。
- OldSpace：老年代（Old Generation），存放年轻代多次回收仍存活的对象会被复制到该空间，根据场景混合多种算法进行内存回收。
- HugeObjectSpace：大对象空间，使用单独的region存放一个大对象的空间。
- ReadOnlySpace：只读空间，存放运行期间的只读数据。
- NonMovableSpace：不可移动空间，存放不可移动的对象。
- SnapshotSpace：快照空间，转储堆快照时使用的空间。
- MachineCodeSpace：机器码空间，存放程序机器码。

[相关参数](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/gc-introduction-V5#相关参数) 

# GC流程 

![GC流程](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250217164523.72910074441006304048982317818118:50001231000000:2800:3243DE1D3DF52669FDB74C28FCE06EF2CC40CC8C14D32372A1447817F315B215.png?needInitFileName=true?needInitFileName=true) 

## HPP GC的类型

**Young GC**

- **触发机制：** 年轻代GC触发阈值在2MB-16MB变化,根据分配速度和存活率等会变化。
- **说明：** 主要回收semi space新分配的年轻代对象。
- **场景：** 前台场景
- **日志关键词：** [ HPP YoungGC ]

**Old GC**

- **触发机制：** 老年代GC触发阈值在20MB-300多MB变化，大部分情况，第一次Old GC的阈值在20M左右，之后会根据对象存活率，内存占用大小进行阈值调整。
- **说明：** 对年轻代和部分老年代空间做整理压缩，其他空间做sweep清理。触发频率比年轻代GC低很多，由于会做全量mark，因此GC时间会比年轻代GC长，单次耗时约5ms~10ms。
- **场景：** 前台场景
- **日志关键词：**[ HPP OldGC ]

**Full GC**

- **触发机制：** 不会由内存阈值触发。应用切换后台之后，如果预测能回收的对象尺寸大于2M会触发一次Full GC。DumpHeapSnapshot 和 AllocationTracker 工具默认会触发Full GC。Native 接口和JS/TS 也有接口可以触发。
- **说明：** 会对年轻代和老年代做全量压缩，主要用于性能不敏感场景，最大限度回收内存空间。
- **场景：** 后台场景
- **日志关键词：**[ CompressGC ]

此后的Smart GC或者 IDLE GC 都是在上述三种GC中做选择。

# SharedHeap 

![结构](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250217164523.68381099713333689335536375499786:50001231000000:2800:C6F6AA4D010BB046E3F35E7C366F0A2380D9F943B3A87FF028013DA8EC13C99B.png?needInitFileName=true?needInitFileName=true) 

- SharedOldSpace：共享老年代空间（这里并不区分年轻代老年代），存放一般的共享对象。
- SharedHugeObjectSpace：共享大对象空间，使用单独的region存放一个大对象的空间。
- SharedReadOnlySpace：共享只读空间，存放运行期间的只读数据。
- SharedNonMovableSpace：共享不可移动空间，存放不可移动的对象。

> [!info] SharedHeap主要用于线程间共享使用的对象，提高效率并节省内存的产物。共享堆并不单独属于某个线程，保存具有共享价值的对象，存活率会更高，去除了SemiSpace的类型。

# Smart GC 

![交互流程](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250217164523.21223882041156809387275032319171:50001231000000:2800:02A7CC5FDD22E057ED3483D6B99C8E0080C417A49EA951AD43DB05EF8278334D.png?needInitFileName=true?needInitFileName=true) 

# 日志解释 

## 开启全量日志 

默认情况只在超过40ms情况下打印，开启方法：

```bash
# 设置开启GC全量日志参数，开启参数为0x905d，关闭GC全量日志，设置为默认值为0x105c
hdc shell param set persist.ark.properties 0x905d
# 重启生效
hdc shell reboot
```

## 日志解读 

```bash

// GC前对象实际占用大小（region实际占用大小）->GC后对象实际占用大小（region实际占用大小），总耗时（+concurrentMark耗时），GC触发原因。
C03F00/ArkCompiler: [gc]  [ CompressGC ] 26.1164 (35) -> 7.10049 (10.5) MB, 160.626(+0)ms, Switch to background
// GC运行时的各种状态以及应用名称
C03F00/ArkCompiler: [gc] IsInBackground: 1; SensitiveStatus: 0; OnStartupEvent: 0; BundleName: com.huawei.hmos.filemanager;
// GC运行时的各阶段耗时统计
C03F00/ArkCompiler: [gc] /***************** GC Duration statistic: ****************/
C03F00/ArkCompiler: [gc] TotalGC:                 160.626 ms
C03F00/ArkCompiler: Initialize:              0.179   ms
C03F00/ArkCompiler: Mark:                    159.204 ms
C03F00/ArkCompiler: MarkRoots:               6.925   ms
C03F00/ArkCompiler: ProcessMarkStack:        158.99  ms
C03F00/ArkCompiler: Sweep:                   0.957   ms
C03F00/ArkCompiler: Finish:                  0.277   ms
// GC后各个部分占用的内存大小
C03F00/ArkCompiler: [gc] /****************** GC Memory statistic: *****************/
C03F00/ArkCompiler: [gc] AllSpaces        used:  7270.9KB     committed:   10752KB
C03F00/ArkCompiler: ActiveSemiSpace  used:       0KB     committed:     256KB
C03F00/ArkCompiler: OldSpace         used:  4966.9KB     committed:    5888KB
C03F00/ArkCompiler: HugeObjectSpace  used:    2304KB     committed:    2304KB
C03F00/ArkCompiler: NonMovableSpace  used:       0KB     committed:    2304KB
C03F00/ArkCompiler: MachineCodeSpace used:       0KB     committed:       0KB
C03F00/ArkCompiler: HugeMachineCodeSpace used:       0KB     committed:       0KB
C03F00/ArkCompiler: SnapshotSpace    used:       0KB     committed:       0KB
C03F00/ArkCompiler: AppSpawnSpace    used: 4736.34KB     committed:    4864KB
C03F00/ArkCompiler: [gc] Anno memory usage size:  45      MB
C03F00/ArkCompiler: Native memory usage size:2.99652 MB
C03F00/ArkCompiler: NativeBindingSize:       0.577148KB
C03F00/ArkCompiler: ArrayBufferNativeSize:   0.0117188KB
C03F00/ArkCompiler: RegExpByteCodeNativeSize:0.280273KB
C03F00/ArkCompiler: ChunkNativeSize:         19096   KB
C03F00/ArkCompiler: [gc] Heap alive rate:         0.202871
// 该虚拟机的此类型GC的整体统计
C03F00/ArkCompiler: [gc] /***************** GC summary statistic: *****************/
C03F00/ArkCompiler: [gc] CompressGC occurs count  6
C03F00/ArkCompiler: CompressGC max pause:    2672.33 ms
C03F00/ArkCompiler: CompressGC min pause:    160.626 ms
C03F00/ArkCompiler: CompressGC average pause:1076.06 ms
C03F00/ArkCompiler: Heap average alive rate: 0.635325
```

- gc类型：[HPP YoungGC]、[HPP OldGC]、[CompressGC]、[SharedGC]。
- TotalGC: 总耗时。其下相应为各个阶段对应的耗时，基本的包括Initialize、Mark、MarkRoots、ProcessMarkStack、Sweep、Finish，实际根据不同的GC流程不同会有不同的阶段。
- IsInBackground：是否在后台场景，1：为后台场景，0：非后台场景。
- SensitiveStatus：是否为敏感场景，1：为敏感场景，0：非敏感场景。
- OnStartupEvent：是否为冷启动场景，1：为冷启动场景，0：非冷启动场景。
- used：当前已分配的对象实际占用的内存空间大小。
- committed：当前实际分配给heap内存空间大小。因为各个空间是按region进行分配的，而region一般也不会被对象完全占满，因此committedSize大于等于usedSize，hugeSpace是会完全相等，因为其一个对象单独占一个region。
- Anno memory usage size：当前进程所有堆申请的内存大小，包括heap与sharedHeap。
- Native memory usage size：当前进程所申请的Native内存大小。
- NativeBindingSize：当前进程堆内对象绑定的Native内存大小。
- ArrayBufferNativeSize：当前进程申请的数组缓存Native内存大小。
- RegExpByteCodeNativeSize：当前进程申请的正则表达式字节码Native内存大小。
- ChunkNativeSize：当前进程申请的ChunkNative内存大小。
- Heap alive rate：堆内对象的存活率。

[文档链接](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/gc-introduction-V5#典型日志) 

## [调试接口](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/gc-introduction-V5#gc开发者调试接口) 

