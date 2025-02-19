---
author: zjmantou
title: ArkTS语言
time: 2025-02-19 周三
tags:
  - 鸿蒙
  - 笔记
---
TypeScript是在JavaScript基础上通过**添加类型定义扩展**而来的，而ArkTS则是TypeScript的进一步扩展。 

ArkTS：
- 低运行时开销，对TS动态类型特性严格限制，提高执行效率；

ESObject：用以标注JS/TS对象的类型。 

# ArkTS与TS区别

[参考文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/typescript-to-arkts-migration-guide-V5) 

- var -> let
- 静态类型，禁止 any
- 禁止运行时变更对象属性
- 对象属性名必须合法，禁止String和纯数字

