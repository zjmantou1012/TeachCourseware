---
author: zjmantou
title: ArkUI框架
time: 2025-02-17 周一
tags:
  - 鸿蒙
  - 技术
---
# 生命周期 

![页面及组件生命周期](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250213113535.89466154540883457558190218673733:50001231000000:2800:4FCC814E0722F5BB92DF71F2F27BC86A860F4663BE7048093581FCAA8525F0B5.png?needInitFileName=true?needInitFileName=true)

[页面及自定义组件生命周期](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-page-custom-components-lifecycle-V5) 

# 自定义布局 

- [onMeasureSize](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/ts-custom-component-layout-V5#onmeasuresize10)：组件每次布局时触发，计算子组件的尺寸，其执行时间先于onPlaceChildren。    
- [onPlaceChildren](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/ts-custom-component-layout-V5#onplacechildren10)：组件每次布局时触发，设置子组件的起始位置。

# @LocalBuilder
子组件引用父组件的@LocalBuilder函数，传入的参数为状态变量，状态变量的改变不会引发@LocalBuilder方法内的UI刷新，原因是@Localbuilder装饰的函数绑定在父组件上，状态变量刷新机制是刷新本组件以及其子组件，对父组件无影响，故无法引发刷新。若使用@Builder修饰则可引发刷新，**原因是@Builder改变了函数的this指向，此时函数被绑定到子组件上，故能引发UI刷新**。

使用场景：

组件Child将@State修饰的label值按照函数传参方式传递到Parent的@Builder和@LocalBuilder函数内，在被@Builder修饰的函数内，this指向Child，参数变化能引发UI刷新，在被@LocalBuilder修饰的函数内，this指向Parent，参数变化不能引发UI刷新。

```ts
class LayoutSize {
  size:number = 0;
}

@Entry
@Component
struct Parent {
  label:string = 'parent';
  @State layoutSize:LayoutSize = {size:0};

  @LocalBuilder
  // @Builder
  componentBuilder($$:LayoutSize) {
    Text(`${'this :'+this.label}`);
    Text(`${'size :'+$$.size}`);
  }

  build() {
    Column() {
      Child({contentBuilder: this.componentBuilder });
    }
  }
}

@Component
struct Child {
  label:string = 'child';
  @BuilderParam contentBuilder:((layoutSize: LayoutSize) => void);
  @State layoutSize:LayoutSize = {size:0};

  build() {
    Column() {
      this.contentBuilder({size: this.layoutSize.size});
      Button("add child size").onClick(()=>{
        this.layoutSize.size += 1;
      })
    }
  }
}
```

[@LocalBuilder和@Builder区别说明](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-localbuilder-V5#localbuilder和builder区别说明)

# @BuilderParam

[尾随闭包](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-builderparam-V5#尾随闭包初始化组件)

## [wrapBuilder：封装全局@Builder](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-wrapbuilder-V5)

当开发者在一个struct内使用了多个全局@Builder函数，来实现UI的不同效果时，多个全局@Builder函数会使代码维护起来非常困难，并且页面不整洁。此时，开发者可以使用wrapBuilder来封装全局@Builder。

[@Builder方法赋值给变量在UI语法中使用](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-wrapbuilder-V5#builder方法赋值给变量在ui语法中使用)

# 样式

## @Styles VS @Extend
- @Styles：重用组件样式
	- 不支持传值
	- 支持全局定义和组件内定义
- @Extend：扩展组件样式
	- 支持传值，参数支持function、event句柄、状态变量
	- 仅支持全局定义，只能在当前文件定义，不能使用export

## @Styles VS stateStyles

@Styles仅支持静态页面复用，stateStyles可以依据组件的内部状态的不同，stateStyles时属性方法。

有五种状态：
- focused：获焦态。
- normal：正常态。
- pressed：按压态。
- disabled：不可用态。
- selected10+：选中态。

# 动画属性@AnimatableExtend

- 仅支持全局定义
- 参数必须为number或者实现`AnimatableArithmetic<T>`接口的自定义类型
- 函数体内只能调用@AnimatableExtend括号内组件的属性方法。
# AnimatableArithmetic接口说明

对非number类型的数据（如数组、结构体、颜色等）做动画，需要实现`AnimatableArithmetic<T>`接口中加法、减法、乘法和判断相等函数. 

- plus
- subtract
- multiply
- equals

# 组件复用@Reusable

记为@Reusable的自定义组件从组件树上被移除时，组件和其对应的JSView对象都会被放入复用缓存中，后续创建新自定义组件节点时，会复用缓存区中的节点，节约组件重新创建的时间。

- 仅用于自定义组件
- ComponentContent不支持传入@Reusable装饰器装饰的自定义组件
- @Reusable装饰器不支持嵌套使用，存在增加内存和不方便维护的问题；

## 使用场景

- 列表滚动
- 动态布局更新：频繁的进行布局更新
- 频繁创建和销毁数据项的视图