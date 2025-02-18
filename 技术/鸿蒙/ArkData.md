---
author: zjmantou
title: ArkData
time: 2025-02-18 周二
tags:
  - 鸿蒙
  - 技术
---
![数据管理架构图](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250217164515.62190833109183913066728997398877:50001231000000:2800:A7D1D8B10528740C72328E674F4026849DCBE7697C7E5A833C3193497263DC01.jpg?needInitFileName=true?needInitFileName=true)

- **用户首选项（Preferences）**:轻量级、持久化、订阅，不支持分布式同步，保存应用配置信息、用户偏好设置。
- **键值型数据管理（KV-Store）**
- **关系型数据管理（RelationalStore）**
- **分布式数据对象（DataObject）**：独立提供对象型结构数据的分布式能力
- **跨应用数据管理（DataShare）**：provider/consumer模型
- **统一数据管理框架（UDMF）**：跨应用、跨设备数据交互标准
	- 标准化数据类型（Uniform Type Descriptor，简称UTD）
	- 标准化数据结构
- **数据管理服务（DatamgrService）**：同步和跨应用共享能力，包括RelationalStore和KV-Store跨设备同步，DataShare静默访问provider数据，暂存DataObject同步对象数据等。

# 标准化数据类型

- 按物理分类，跟节点为general.entity，如文件、目录等
- 按逻辑分类，跟节点为general.object，描述功能性特征，如图片、网页等

![物理标准化数据类型示意图](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250217164515.15265213051536130086339177386801:50001231000000:2800:C05C00F1A1BA2D3ED2C37DCE261B5CA4D7FA0E17124CDF801D7F264A22D4C51A.png?needInitFileName=true?needInitFileName=true) 

![逻辑标准化数据类型示意图](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250217164515.12713797064435744170663728853488:50001231000000:2800:89E50D16E1B6C2BC23B77D24FB5CD7877764C985A0C42EA0C3EBEFBB82B34E85.png?needInitFileName=true?needInitFileName=true) 

## 预置数据类型 

如“general.audio”、“general.video”，参考[UTD预置列表](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/uniform-data-type-list-V5)。 

# 标准化数据结构

| 数据结构                                                                                                                                            | 数据类型                   | 说明  |
| :---------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------- | :-- |
| [PlainText](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-data-uniformdatastruct-V5#plaintext)                   | 'general.plain-text'   | 纯文本 |
| [Hyperlink](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-data-uniformdatastruct-V5#hyperlink)                   | 'general.hyperlink'    | 超链接 |
| [HTML](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-data-uniformdatastruct-V5#html)                             | 'general.html'         | 富文本 |
| [OpenHarmonyAppItem](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-data-uniformdatastruct-V5#openharmonyappitem) | 'openharmony.app-item' | 图标  |

# Preferences

![用户首选项运作机制](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250217164516.75319919205630400427310567704917:50001231000000:2800:15771E032DB67D327162401712A12F8736E00FCA6797BC2B2AD4A51A251761E8.jpg?needInitFileName=true?needInitFileName=true)

- 无法保证并发、不支持多进场场景
- Key为string，不超过1024个字节
- value为string的时候使用UTF-8，不超过16M
- 非UTF-8数据使用Uint8Array类型存储
- removePreferencesFromCache和deletePreferences，订阅的数据取消订阅
- deletePreferences不允许多线程多进程调用
- 轻量级，建议不超过一万条

[开发示例](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/data-persistence-by-preferences-V5#开发步骤)

# KV-Store

- 设备协同数据库，Key的长度≤896 Byte，Value的长度<4 MB
- 单版本数据库，Key的长度≤1 KB，Value的长度<4 MB
- 每个应用程序最多支持同时打开16个键值型分布式数据库。
- 键值型数据库事件回调方法中不允许进行阻塞操作，例如修改UI组件。

[开发示例](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/data-persistence-by-kv-store-V5#开发步骤) 

# RelationalStore

基于SQLite组件。 

- 单次查询数据量不超过5000条。
- 在[TaskPool](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-taskpool-V5)中查询。
- 拼接SQL语句尽量简洁。
- 合理地分批次查询。

![运行机制](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250217164516.91410210851764176215962734107449:50001231000000:2800:4C28DCC5B0FE6F48F53C5B7578B4758F2E3CD63FB8A2437B1EDB34689FAD0650.jpg?needInitFileName=true?needInitFileName=true) 

限制：
- 系统默认日志方式是WAL（Write Ahead Log）模式，系统默认落盘方式是FULL模式。
- 数据库中有4个读连接和1个写连接，线程获取到空闲读连接时，即可进行读取操作。当没有空闲读连接且有空闲写连接时，会将写连接当做读连接来使用。
- 为保证数据的准确性，数据库同一时间只能支持一个写操作。
- 当应用被卸载完成后，设备上的相关数据库文件及临时文件会被自动清除。
- ArkTS侧支持的基本数据类型：number、string、二进制类型数据、boolean。
- 为保证插入并读取数据成功，建议一条数据不要超过2M。超出该大小，插入成功，读取失败。

[开发示例](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/data-persistence-by-rdb-store-V5#开发步骤) 

# 分布式（DataObject）

## 键值型数据库跨设备同步

![数据跨设备同步机制](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250217164516.16150052666022922128834177908537:50001231000000:2800:50E6BAF5B687EA40204816BCCA564E93EDCDDFD375A4DB5491C96A6F17E0727A.jpg?needInitFileName=true?needInitFileName=true)

[开发示例](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/data-sync-of-kv-store-V5#开发步骤) 

## 关系型数据库跨设备数据同步 

![数据跨设备同步机制](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250217164517.89363133878976104623064957250716:50001231000000:2800:2DF842D6EF17AE788BCBCA5994419136730160CDABC9F4A14CBF81DA1A0D61BA.jpg?needInitFileName=true?needInitFileName=true) 

[开发示例](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/data-sync-of-rdb-store-V5#开发步骤) 

## 分布式数据对象跨设备数据同步

![原理](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250217164517.02695226217410416702681059294619:50001231000000:2800:C8581A53725751E83961EF0E2417B30B0D444CB6783A206BB406A1055E1676A5.jpg?needInitFileName=true?needInitFileName=true) 

[开发示例](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/data-sync-of-distributed-data-object-V5#开发步骤) 

