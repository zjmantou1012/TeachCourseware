---
author: zjmantou
title: Android艺术之画一条平滑的曲线
time: 2024-07-04 周四
tags:
  - Android
  - 资料
  - UI
---


# 前言

说的是曲线，其实想法是来自一个曲线图的需求。图表这种东西，项目开发中也不少见，大多情况找个通用的开源框架改改就得了（老板们别打我），然而通用赶不上脑洞，要做交互和视觉比较特别的图表时，还是自己造一个轮子比较靠谱，这次要研究的就是一个优雅而平滑的曲线怎么画出来。

# 实现方法分析

曲线图的责任概括起来就是把数据输出为对应的图像，我们这次需求的目标效果图是这样的：

![](https://nos.netease.com/cloud-website-bucket/2018080912501354cd4d8a-59a1-4062-9fe2-6dd68f5eaefa.png)

坐标轴和指示线等功能不是这篇文章的重点，抛开它们先不讨论，这次研究的重点在于曲线的绘制，而说到绘制曲线，最常用的参数曲线函数就是贝塞尔曲线。

**二次贝塞尔曲线：**

![](https://nos.netease.com/cloud-website-bucket/201808091250518debc724-14c0-4d0c-be8e-c1911e7d872c.png)

![](https://nos.netease.com/cloud-website-bucket/20180809125025866a5542-bc31-442b-a74f-5a04f56d62cc.gif)

**三次贝塞尔曲线：**

![](https://nos.netease.com/cloud-website-bucket/2018080912510267627f6f-f0db-4445-960e-3ec0e200a966.png)

![](https://nos.netease.com/cloud-website-bucket/20180809125102784558af-27e6-4b83-933f-3039c6cb72da.gif)

关于贝塞尔曲线更详细的内容戳[这里](https://zh.wikipedia.org/wiki/%E8%B2%9D%E8%8C%B2%E6%9B%B2%E7%B7%9A)，Android也提供了绘制贝塞尔曲线的方法，方法参数就是对应贝塞尔曲线的控制点：

```java
// 二次贝塞尔曲线
path.quadTo(auxiliaryX, auxiliaryY, endPointX, endPointY);
canvas.drawPath(path, paint);
// 三次贝塞尔曲线
path.cubicTo(auxiliaryOneX, auxiliaryOneY, auxiliaryTwoX, auxiliaryTwoY, endPointX, endPointY);
canvas.drawPath(path, paint);
```

仔细分析目标效果图，我们可以把曲线图拆分为一段段短的曲线，每两个数据点之间用三次贝塞尔曲线来绘制，保证曲线经过每个数据点，如下图所示，红色和蓝色的线段分别是两条三次贝塞尔曲线。

![](https://nos.netease.com/cloud-website-bucket/20180809125129c51339ff-3743-4e54-8381-bc54c51356ba.png)

这样我们的任务就变为确定每一段三次贝塞尔曲线四个控制点P0、P1、P2和P3的位置，毫无疑问，数据点可以作为曲线的起点P0和终点P3，某个数值点既是前一条曲线的终点也是后一条曲线的起点，而确定剩下的P1和P2的位置还需要一个重要的课堂知识：

光滑曲线必定处处可导。 —— 《高中数学·必修一》

要保证整条曲线图是光滑的，关键在于贝塞尔曲线的连接点也要保证光滑，而三次贝塞尔曲线中P0和P1的连线就是P0点的切线，P2和P3的连线是P3的切线，所以我们只要保证数据点左右控制点共线，就可以保证曲线在数据点处是可导光滑的。![](https://nos.netease.com/cloud-website-bucket/201808091251418ce5da5d-11ac-43c9-83c9-2ca6756389d8.png)

如图所示，A0、A3、B3分别是三个数据点，A1、A2、B1、B2是控制点，A0、A1、A2和A3构建了一条三次贝塞尔曲线，B0、B1、B2和B3也构建了一条三次贝塞尔曲线，当A2和B1共线时，A2和B1的连线就是A3点的切线，数据点A3点就是可导光滑的。之后，只要我们使控制点A2、B1连线的斜率和A3左右数据点A0、B3连线的斜率保持一致，就可以让曲线的效果更自然。

设A0的坐标为（A0X，A0Y），A3的坐标为（A3X，A3Y），B3的坐标为（B3X，B3Y），控制点A2、B1的坐标计算方法如下：

```
令
A0和B3连线的斜率 k = (B3Y - A0Y) / (B3X - A0X)
常数 b = A3Y - k * A3X
则
A2的X坐标 A2X = A3X - (A3X - A0X) * rate
A2的Y坐标 A2Y = k * A2X + b
B1的X坐标 B1X = A3X + (B3X - A3X) * rate
B1的Y坐标 B1Y = k * B1X + b
```

rate是一个（0, 0.5）区间内的值，数值越大，数值点之间的曲线弧度越小。 除此以外，如果数值点是第一个点或者最后一个点，可以把斜率k视为0，然后只计算左控制点或者有控制点。 我们只要把每个数值点左右的控制点坐标计算出来，然后画出每一段曲线，就可以组成一个完整的圆滑曲线了。

# 代码实现

基本原理就是这么多，还是贴代码实际。先计算全部数据点的坐标，用mValuePointList保存起来，max是图表显示的最大值，scaleX和scaleY分别是单位长度

```kotlin
private fun calculateValuePoint(itemList: List<Item>, max: Float, scaleX: Float, scaleY: Float) {
    mValuePointList.clear()
    for ((i, item) in itemList.withIndex()) {
        val x = i * scaleX
        val y = (max - item.value) * scaleY
        mValuePointList.add(PointF(x, y))
    }
}
```

然后计算控制点的坐标，用mControlPointList保存起来

```kotlin
private fun calculateControlPoint(pointList: List<PointF>) {
    mControlPointList.clear()
    if (pointList.size <= 1) {
        return
    }
    for ((i, point) in pointList.withIndex()) {
        when (i) {
            0 -> {//第一项
                //添加后控制点
                val nextPoint = pointList[i + 1]
                val controlX = point.x + (nextPoint.x - point.x) * SMOOTHNESS
                val controlY = point.y
                mControlPointList.add(PointF(controlX, controlY))
            }
            pointList.size - 1 -> {//最后一项
                //添加前控制点
                val lastPoint = pointList[i - 1]
                val controlX = point.x - (point.x - lastPoint.x) * SMOOTHNESS
                val controlY = point.y
                mControlPointList.add(PointF(controlX, controlY))
            }
            else -> {//中间项
                val lastPoint = pointList[i - 1]
                val nextPoint = pointList[i + 1]
                val k = (nextPoint.y - lastPoint.y) / (nextPoint.x - lastPoint.x)
                val b = point.y - k * point.x
                //添加前控制点
                val lastControlX = point.x - (point.x - lastPoint.x) * SMOOTHNESS
                val lastControlY = k * lastControlX + b
                mControlPointList.add(PointF(lastControlX, lastControlY))
                //添加后控制点
                val nextControlX = point.x + (nextPoint.x - point.x) * SMOOTHNESS
                val nextControlY = k * nextControlX + b
                mControlPointList.add(PointF(nextControlX, nextControlY))
            }
        }
    }
}
```

最后绘制曲线和数值

```kotlin
//连接各部分曲线
mPath.reset()
val firstPoint = pointList.first()
mPath.moveTo(firstPoint.x, height)
mPath.lineTo(firstPoint.x, firstPoint.y)
for (i in 0 until pointList.size * 2 step 2) {
    val leftControlPoint = controlPointList[i]
    val rightControlPoint = controlPointList[i + 1]
    val rightPoint = pointList[i / 2 + 1]
    mPath.cubicTo(leftControlPoint.x, leftControlPoint.y, rightControlPoint.x, rightControlPoint.y, rightPoint.x, rightPoint.y)
}
val lastPoint = pointList.last()
//填充渐变色
mPath.lineTo(lastPoint.x, height)
mPath.lineTo(firstPoint.x, height)
mPaint.alpha = 255
mPaint.style = Paint.Style.FILL
mPaint.shader = LinearGradient(0F, 0F, 0F, height, COLOR_GRAPH_FILL, null, Shader.TileMode.CLAMP)
canvas.drawPath(mPath, mPaint)
//绘制全部路径
mPath.setLastPoint(lastPoint.x, height)
mPaint.strokeWidth = SIZE_GRAPH
mPaint.style = Paint.Style.STROKE
mPaint.shader = null
mPaint.color = COLOR_GRAPH
canvas.drawPath(mPath, mPaint)
for (i in 0..pointList.size()) {
    val point = pointList[i]
    //画数值线
    mPaint.color = COLOR_POINT
    mPaint.alpha = 100
    canvas.drawLine(point.x, point.y, point.x, height, mPaint)
    //画数值点
    mPaint.style = Paint.Style.FILL
    mPaint.alpha = 255
    canvas.drawCircle(point.x, point.y, SIZE_POINT, mPaint)
}
```

OK！大功告成，最终效果图：

![](https://nos.netease.com/cloud-website-bucket/20180809125202f49c8a97-ddbc-4871-8566-81e41a36759c.png)

剩下的刻度效果和滑动效果并不是太复杂，有时间再写一篇吧，谢谢各位看官支持。


# 原文链接
[Android艺术之画一条平滑的曲线](https://sq.sf.163.com/blog/article/185870408682938368)
