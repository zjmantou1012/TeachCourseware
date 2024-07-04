---
author: zjmantou
title: 自带美感的贝塞尔曲线原理与实战——Android高级UI
time: 2024-07-04 周四
tags:
  - Android
  - 资料
  - UI
---


> 目录
> 
> 一、前言
> 
> 二、贝塞尔曲线的绘制规则
> 
> 三、在canvas中如何绘制贝塞尔曲线
> 
> 四、实战
> 
> 五、写在最后

## 一、前言

贝塞尔曲线，想必大家或多或少都听过这个词，因为其控制简单，且其曲线更符合我们大众的审美，所以在很多领域都有涉及，当然这些都不是我们今天要进行讨论和分享的重点。今天要分享的是如何成为自定义UI中的一把利器，先上两张图看看效果，然后开始我们的分享。

**圆变心效果图** ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e0815a558b~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

**乘风破浪的小船** ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e089707b3b~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

> 文末会给出源码，勿急勿急，弄懂原理，才能面对一切需求

## 二、贝塞尔曲线的绘制规则

想要讲清楚多阶贝塞尔曲线，我们先要从一阶开始讲起。我先来看下**一阶贝塞尔曲线**的动态图。

> 动画demo的源码，请移步[github传送门](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FzincPower%2FUI2018%2Fblob%2Fe69783a81a6f387a5970327e4c0905ff943e1da7%2Fclass7_bezier%2Fsrc%2Fmain%2Fjava%2Fcom%2Fzinc%2Fclass7_bezier%2Fwidget%2FBezierView.java "https://github.com/zincPower/UI2018/blob/e69783a81a6f387a5970327e4c0905ff943e1da7/class7_bezier/src/main/java/com/zinc/class7_bezier/widget/BezierView.java")

### 1、一阶贝塞尔曲线

**一阶贝塞尔曲线解析：** 两点控制一条直线，只是刚好一阶的控制点是静止的，所以当比例增加时(图中左下角的u值即为比例，比例的值 u = 左上的点到移动的小红点的距离 / 整条线的长度)，贝塞尔曲线上的点（即小红点）一直都是在一条静态的线上挪动，致使**一阶贝塞尔曲线的结果就是一条直线，只是和两个控制点连接的直线重合了。**

**看到这你可能还有些云里雾里，切勿心急，顺着往下看，你会恍然大悟。**

![一阶贝塞尔曲线](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e087b19649~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

### 2、二阶贝塞尔曲线

看完一阶的，可能各位童鞋还没感受到其魅力。我们进行升阶，升至二阶。**二阶贝塞尔曲线**动态效果如下图：

![二阶贝塞尔曲线动态图](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e089389ffa~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

**二阶贝塞尔曲线解析：** 我们需要借助以下的一张静态图来讲解 ![二阶贝塞尔曲线](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e07f458b75~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp) **第一步：** 先看**二阶的控制点和基线**（蓝色的点和线），然后按照**比例值u**，从 **AB** 和 **BC** 上取 **D** 和 **E**。具体为

```java
AD/AB = BE/BC = u(比例值)
```

**第二步：** 从第一步得出了 **D** 和 **E** 两个点，这两个点便是 **一阶的控制点** （黄色点），将DE连起便是 **一阶的基线** （黄色线），然后按照和 **第一步一样** 的 **比例值u**，从 **DE** 上取 **F**。具体为

```java
DF/DE = u(比例值)
```

**第三步：** 从第二步得出的点 **F** 就是 **最终贝塞尔曲线**（红色线）上当比例值为u时的点。 **当比例值u 从零到一变化时，D和E 在 AB和BC 上进行移动，从而让 DE 直线会被“推动”起来。而 DE 直线上的点 F 也因u的变化而“移动起来”，这一连带的推动，最终产生了 点F “走”过的轨迹（红色线）便是最终的贝塞尔曲线。**

> 值得一说： 贝塞尔曲线的绘制，无论多少阶（一阶除外），均需要进行降阶，降至一阶。在 “二阶贝塞尔曲线解析” 这段文字中，从 第一步 到 第二步 的过程就是在降阶。

#### 从上面的三步中，我们可以得出如下三条结论：

- 一阶的基线从二阶得来，二阶的基线从三阶得来（待会会继续讲三阶，稍安勿躁），推而广之，**n阶的基线便从（n+1）阶得来**；
- 除了最高阶的控制点是固定的，降阶过程中的控制点**全都**是按 比例值u 进行取点。所以在二阶的例子中，我们可以得出以下这样一个等式

```java
AD/AB = BE/BC = DF/DE = u(比例值)
```

- 贝塞尔曲线最终的路径是由 **一阶基线** 上的游走的红色小点形成的；

### 3、三阶贝塞尔曲线

理解完二阶，童鞋们大多能根据上面的 **三条结论** 得出如何绘制三阶贝塞尔曲线，带着你心中的猜想，我们继续解析三阶贝塞尔曲线，先来看下**三阶贝塞尔曲线**的动态效果图：

![三阶贝塞尔曲线动态图](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e0895ca16c~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

**三阶贝塞尔曲线解析：** 按照二阶的惯例，为了方便理解，我们还是使用一张静态图，以下便是三阶贝塞尔曲线的静态图（稍微凌乱了些，可以根据颜色进行区分）

![三阶贝塞尔曲线](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e0b06a4076~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

**第一步：** **三阶的基线** 为 **AB**、**BC**、**CD**（蓝色线） ，然后按照 **比例值u** 分别取 **E**、**F**、**G** ；

**第二步：** 从第一步便得出 二阶的控制点 **E**、**F**、**G**（黄色点），连线而得 **EF**、**FG** 两条二阶基线，同样按照 **比例值u** 分别取 **H** 和 **I** ；

**第三步：** 从第二步得出了 一阶的控制点 **H** 和 **I** （绿色点），连线而得 **HI** 一阶基线，按照 **比例值u** 取得 **J**，就是最终的贝塞尔曲线，当比例值u为0.55时，所在的点。

#### 值得一提

从三阶我们可以知道所有点的比例值都是一样的，具体如下

```java
AE/AB = BF/BC = CG/CD = EH/EF = FI/FG = HJ/HI = 比例值u
```

### 4、七阶贝塞尔曲线

前面看了一二三阶的贝塞尔曲线，想必童鞋们已经知道这绘制的规律。接下来我们看看 **七阶贝塞尔曲线** 的动态图，其规则和三阶是一样的，都是从七阶降至六阶再到五阶等等，这里就不再赘述。

![7阶贝塞尔曲线](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e0b5f6cd8f~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

> 动画demo的源码，请移步[github传送门](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FzincPower%2FUI2018%2Fblob%2Fe69783a81a6f387a5970327e4c0905ff943e1da7%2Fclass7_bezier%2Fsrc%2Fmain%2Fjava%2Fcom%2Fzinc%2Fclass7_bezier%2Fwidget%2FBezierView.java "https://github.com/zincPower/UI2018/blob/e69783a81a6f387a5970327e4c0905ff943e1da7/class7_bezier/src/main/java/com/zinc/class7_bezier/widget/BezierView.java")

## 三、在canvas中如何绘制贝塞尔曲线

### 1、二阶贝塞尔曲线

二阶贝塞尔曲线 在 Path 类中有提供现成的 API

```java
public void quadTo(float x1, float y1, float x2, float y2)
```

#### 如何使用？

我们借助上面的二阶贝塞尔曲线的静态图，进行讲解。 ![二阶贝塞尔曲线](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e07f458b75~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp) 假设我们要绘制图中的这条红色的二阶贝塞尔曲线，只需进行如下代码操作

```java
// 初始化 路径对象
Path path = new Path();
// 移动至第一个控制点 A(ax,ay)
path.moveTo(ax, ay);
// 填充二阶贝塞尔曲线的另外两个控制点 B(bx,by) 和 C(cx,cy)，切记顺序不能变
path.quadTo(bx, by, cx, cy);
// 将 贝塞尔曲线 绘制至画布
canvas.drawPath(path, paint);
```


这段代码绘制的效果和图中是一样的，我就不在贴图了。

### 2、三阶贝塞尔曲线

很幸运的是，三阶贝塞尔曲线 在 Path 类中也有提供现成的 API

```java
public void cubicTo(float x1, float y1, float x2, float y2, float x3, float y3)
```

我们借助上面 三阶贝塞尔曲线静态图 进行讲解。 ![三阶贝塞尔曲线](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e0b732f726~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp) 假设我们要绘制图中的这条红色的三阶贝塞尔曲线，只需进行如下代码操作

```java
// 初始化 路径对象
Path path = new Path();
// 移动至第一个控制点 A(ax,ay)
path.moveTo(ax, ay);
// 填充三阶贝塞尔曲线的另外三个控制点：
// B(bx,by) C(cx,cy) D(dx,dy) 切记顺序不能变
path.cubicTo(bx, by, cx, cy, dx, dy);
// 将 贝塞尔曲线 绘制至画布
canvas.drawPath(path, paint);

```

### 3、多阶贝塞尔曲线

看完二阶和三阶贝塞尔曲线的使用，是不是觉得非常的简单。但是系统提供的API就止步三阶贝塞尔曲线了，这是因为高阶在实际的开发过程中不是很常用，如果真的需要使用再高阶的贝塞尔曲线，那就只能自己进行降阶了。

我们借助以下的二阶贝塞尔曲线图来推导我们的降阶公式。 ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e0b899de14~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp) 先确定几个坐标 A(ax, ay)、B(bx, by)、C(cx, cy)、D(dx, dy)、E(ex, ey)、F(fx, fy)

当然一开始我们只知道 **A、B、C** 三个点的坐标，所以 **D** 的坐标由 **A、B** 进行求出具体如下

```java
D点的x轴坐标：dx = ax + (bx-ax) * u  = (1-u) * ax + u * bx   (u ∈ [0,1])
D点的y轴坐标：dy = ay + (by-ay) * u  = (1-u) * ay + u * by   (u ∈ [0,1])
```

同理，**E** 的坐标由 **B、C** 进行求出，计算的逻辑完全一样。具体如下

```java
E点的x轴坐标：ex = bx + (cx-bx) * u  = (1-u) * bx + u * cx   (u ∈ [0,1])
E点的y轴坐标：ey = by + (cy-by) * u  = (1-u) * by + u * cy   (u ∈ [0,1])

```

当得出 **D和E** 点，就可以进行求 **点F**，逻辑还是一样。具体如下

```java
F点的x轴坐标：fx = dx + (ex-dx) * u  = (1-u) * dx + u * ex   (u ∈ [0,1])
F点的y轴坐标：fy = dy + (ey-dy) * u  = (1-u) * dy + u * ey   (u ∈ [0,1])

```

至此最终的点 **F** 的可绘制坐标便得出。

#### 推导公式

从以上的 **计算公式** 和 之前的 **“三个结论”**，借助下图我们可以得出一个公式

**P0k = (1-u) * P0k-1 + u * P1k-1**

> Tips: x轴 和 y轴 的坐标计算公式是一样，所以我们这里就使用 x轴 作为代表，方便讲解

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e0bbf887ed~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

由**通用公式**，想必童鞋们已经想到算法中的一个词叫 **“递归”**，的确没错，但细想一下还缺少一个 **递归的终止条件** 。我们再细想一下，其实终止条件就是 **降阶最开始依赖的控制点是固定不变的，或是说是我们程序猿给定的，所以不用计算直接返回该控制点的x轴或y轴即可。**

最终的递归公式如下 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb92c0791cc9484d971d55ab41b716d7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

> 公式说明：
> 
> 1、k 表示阶数，当 k=n 时，即相当于前面demo所讲的一阶控制点；当 k=0 时，表示最高阶的控制点，即我们程序猿最初给定的那几个控制点；
> 
> 2、 i 表示点的下标，这个只是为了便于区分，可参照上面的图进行带入理解；
> 
> 3、u 表示比例值

将通用公式编写成如下代码，调用 buildBezierPoint 方法，即可获得对应的最终的贝塞尔曲线，二阶和三阶也同样适用。

```java
/**
 * 构建贝塞尔曲线，具体点数由 参数frame 决定
 *
 * @param controlPointList 控制点的坐标
 * @param frame            帧数
 * @return
 */
public static List<PointF> buildBezierPoint(List<PointF> controlPointList,
                                            int frame) {
    List<PointF> pointList = new ArrayList<>();

    // 此处注意，要用1f，否则得出结果为0
    float delta = 1f / frame;

    // 阶数，阶数=绘制点数-1
    int order = controlPointList.size() - 1;

    // 循环递增
    for (float u = 0; u <= 1; u += delta) {
        pointList.add(new PointF(BezierUtils.calculatePointCoordinate(BezierUtils.X_TYPE, u, order, 0, controlPointList),
                BezierUtils.calculatePointCoordinate(BezierUtils.Y_TYPE, u, order, 0, controlPointList)));
    }

    return pointList;

}

/**
 * 计算坐标 [贝塞尔曲线的核心关键]
 *
 * @param type             {@link #X_TYPE} 表示x轴的坐标， {@link #Y_TYPE} 表示y轴的坐标
 * @param u                当前的比例
 * @param k                阶数
 * @param p                当前坐标（具体为 x轴 或 y轴）
 * @param controlPointList 控制点的坐标
 * @return
 */
public static float calculatePointCoordinate(@IntRange(from = X_TYPE, to = Y_TYPE) int type,
                                             float u,
                                             int k,
                                             int p,
                                             List<PointF> controlPointList) {

    /**
     * 公式解说：（p表示坐标点，后面的数字只是区分）
     * 场景：有一条线p1到p2，p0在中间，求p0的坐标
     *      p1◉--------○----------------◉p2
     *            u    p0
     *
     * 公式：p0 = p1+u*(p2-p1) 整理得出 p0 = (1-u)*p1+u*p2
     */

    // 一阶贝塞尔，直接返回
    if (k == 1) {

        float p1;
        float p2;

        // 根据是 x轴 还是 y轴 进行赋值
        if (type == X_TYPE) {
            p1 = controlPointList.get(p).x;
            p2 = controlPointList.get(p + 1).x;
        } else {
            p1 = controlPointList.get(p).y;
            p2 = controlPointList.get(p + 1).y;
        }

        return (1 - u) * p1 + u * p2;

    } else {

        /**
         * 这里应用了递归的思想：
         * 1阶贝塞尔曲线的端点 依赖于 2阶贝塞尔曲线
         * 2阶贝塞尔曲线的端点 依赖于 3阶贝塞尔曲线
         * ....
         * n-1阶贝塞尔曲线的端点 依赖于 n阶贝塞尔曲线
         *
         * 1阶贝塞尔曲线 则为 真正的贝塞尔曲线存在的点
         */
        return (1 - u) * calculatePointCoordinate(type, u, k - 1, p, controlPointList)
                + u * calculatePointCoordinate(type, u, k - 1, p + 1, controlPointList);

    }

}

```
## 四、实战

经过漫长的理论，童鞋们早就摩拳搽掌，想用贝塞尔曲线前去挑战设计师，少侠勿急，看完实战我们再去碾压😄。

> 温馨提示：
> 
> 理论是进阶中必不可少的部分，否则只知其然而不知其所以然。永远只能是作为使用别人代码的使用者，而不是创造者，更无法体会到创造的快乐。

贝塞尔曲线Demo的 Github 入口：[传送门](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FzincPower%2FUI2018%2Fblob%2Fe69783a81a6f387a5970327e4c0905ff943e1da7%2Fclass7_bezier%2Fsrc%2Fmain%2Fjava%2Fcom%2Fzinc%2Fclass7_bezier%2Factivity%2FClientActivity.java "https://github.com/zincPower/UI2018/blob/e69783a81a6f387a5970327e4c0905ff943e1da7/class7_bezier/src/main/java/com/zinc/class7_bezier/activity/ClientActivity.java")

### 1、圆变心

文章最开始出现的就是以下这张效果图，现在是时候进行撸起袖子开始打代码了。

#### 效果图

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e0815a558b~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

#### 动画分析：

动态图中，我们可以清楚的看出，从一个圆形慢慢变成心形，然后带有一点弹性效果。这样一分析，我们便需要三样东西：圆、心、弹性效果公式，接下来就是逐个突破。

#### 准备零件

**（1）圆** 此圆非彼圆，我们不能借助canvas直接使用drawCircle进行绘制，因为这样的圆我们**无法控制**。那要如何处理呢？当然是用**贝塞尔曲线画圆**，因为这样一来这个 **“圆”的控制点** 就全都在我们的可控范围内，因为我持有了这些控制点就能进行坐标的变动，进而改变曲线的形状。

正当你在坐等这个 **贝塞尔曲线画圆的公式** 时，我又要泼一盆冷水了，因为根本就不存在这样一个公式。但我们可以通过前面的理论找到一个 **近似圆的贝塞尔曲线公式** 。

> 至于 贝塞尔曲线 为什么无法画出一个圆，有兴趣的童鞋们自行百度和google吧，毕竟这个一两行字无法解释清楚。

我们可以通过 **三阶贝塞尔曲线** 画出一段圆弧，通过四段圆弧就能拼凑出一个完整的圆了。但是又来了一个问题，**三阶贝塞尔曲线有四个控制点，两端的控制点容易取，中间的控制点如何取？** 带着这个疑问，我们来看下面这个动画，当 **控制点比例** 从0到1增加过程中，**蓝色区域从方形慢慢的变得接近圆，然后溢出变成圆角方形**，**红色的圆圈是用canvas的drawCircle绘制**，从一些贝塞尔曲线绘制圆的论证资料和这里的动画效果可以得出，当 **控制点比例等于0.55时(保留两位小数)，最接近一个圆。** 前面提到的 **四段圆弧的贝塞尔曲线** ，在这里使用了四种颜色，需要自己体验效果的童鞋，请进[传送门](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FzincPower%2FUI2018%2Fblob%2Fe69783a81a6f387a5970327e4c0905ff943e1da7%2Fclass7_bezier%2Fsrc%2Fmain%2Fjava%2Fcom%2Fzinc%2Fclass7_bezier%2Fwidget%2FCircleBezierView.java "https://github.com/zincPower/UI2018/blob/e69783a81a6f387a5970327e4c0905ff943e1da7/class7_bezier/src/main/java/com/zinc/class7_bezier/widget/CircleBezierView.java")。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e0bd8f5544~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

可能还有些童鞋对动态图中的 **控制点比例** 不太理解，我们借助下图来解释，图中只留了一段圆弧，其他的三段是一样的道理。具体比例公式如下

控制点比例=ABED=CDAE控制点比例 =\frac{AB}{ED} = \frac{CD}{AE}控制点比例=EDAB​=AECD​

E为圆心，AE和ED为半径，即AE=ED；所以AB=CD;

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e0e07b31cf~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

至此圆的问题解决了。我们继续过关斩将。

**（2）心** 心要如何绘制呢？小盆友这里给出一个小工具，我们可以通过**自行拽动**来获取需要的图形，然后**打印坐标**（单位dp），拿到坐标了就可以为所欲为了。工具效果图如下，我们以拽动一个圆为例：

**拽动的动态效果图** ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e0e0fec0a4~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp) **打印出来的坐标点（单位为dp）：** 坐标肯定会**有些许偏差**，毕竟是手指拽动出来的，而且左右的心是不对称的，所以需要进行微调一半心，然后另一半心进行**对称取坐标**。如此一来，心形也搞定了。 ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e0e923b00e~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

这个小工具可用于从圆变成另一种形状，而不局限于心形，或许说局限于我们的想象力。下图是小盆友随便拽出来的一个图，个人觉得有点像兔子🐰，哈哈，挺抽象的吧。感兴趣的可以进[传送门](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FzincPower%2FUI2018%2Fblob%2Fe69783a81a6f387a5970327e4c0905ff943e1da7%2Fclass7_bezier%2Fsrc%2Fmain%2Fjava%2Fcom%2Fzinc%2Fclass7_bezier%2Fwidget%2FDIYBezierView.java "https://github.com/zincPower/UI2018/blob/e69783a81a6f387a5970327e4c0905ff943e1da7/class7_bezier/src/main/java/com/zinc/class7_bezier/widget/DIYBezierView.java")。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e0ec45ee27~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

**（3）弹性效果公式** 终于到最后一个零件啦，弹性效果公式可以从一个网站尝试得出

[inloop.github.io/interpolato…](https://link.juejin.cn?target=http%3A%2F%2Finloop.github.io%2Finterpolator%2F "http://inloop.github.io/interpolator/")

最终得到一个让自己觉得还不错的公式

```java
float x = (float) animation.getAnimatedValue();
float factor = 0.15f;
double value = Math.pow(2, -10 * x) * Math.sin((x - factor / 4) * (2 * Math.PI) / factor) + 1;

```

#### 组装零件

零件都已经备好了，组装起来就是我们看到的效果，因为代码比较简单，就不再贴出来了，需要的请进[传送门](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FzincPower%2FUI2018%2Fblob%2Fe69783a81a6f387a5970327e4c0905ff943e1da7%2Fclass7_bezier%2Fsrc%2Fmain%2Fjava%2Fcom%2Fzinc%2Fclass7_bezier%2Fwidget%2FHeartView.java "https://github.com/zincPower/UI2018/blob/e69783a81a6f387a5970327e4c0905ff943e1da7/class7_bezier/src/main/java/com/zinc/class7_bezier/widget/HeartView.java")。

### 2、乘风破浪的小船

文章最开始出现的第二个就是以下这张效果图，看完第一个实战代码，童鞋们心中大概有自己的思路了。这里就不再讲细节了，这里只是分享下大概的思路。细节可以自行翻阅代码，代码在关键地方都写了注释，[传送门](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FzincPower%2FUI2018%2Fblob%2Fe69783a81a6f387a5970327e4c0905ff943e1da7%2Fclass8_pathmeasure%2Fsrc%2Fmain%2Fjava%2Fcom%2Fzinc%2Fclass8_pathmeasure%2FBoatWaveView.java "https://github.com/zincPower/UI2018/blob/e69783a81a6f387a5970327e4c0905ff943e1da7/class8_pathmeasure/src/main/java/com/zinc/class8_pathmeasure/BoatWaveView.java")。

#### 效果图

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e089707b3b~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

#### 大致思路

（1）绘制蓝色的波浪、浅蓝色的波浪和小船的轨迹，这里使用的是二阶贝塞尔曲线 （2）将蓝色和浅蓝色的波浪的波浪绘制至canvas，但偏移量不同。 （3）使用 PathMeasure 测量小船轨迹，同时改变的坐标和方向。

这样便完成了这个 “乘风破浪的小船” 效果

> 关于 PathMeasure 怎么使用，感兴趣的童鞋，请入[传送门](https://juejin.im/post/6844903752869085192 "https://juejin.im/post/6844903752869085192")

### 3、粘性小红点

粘性小红点的效果算是见的比较多的，例如QQ的未读消息便是这效果。这里同样也是使用二阶贝塞尔曲线，至于细节同样不给出，代码中注释也很齐全，需要的同学请入[传送门](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FzincPower%2FUI2018%2Fblob%2Fe69783a81a6f387a5970327e4c0905ff943e1da7%2Fclass7_bezier%2Fsrc%2Fmain%2Fjava%2Fcom%2Fzinc%2Fclass7_bezier%2Fwidget%2FStickDotView.java "https://github.com/zincPower/UI2018/blob/e69783a81a6f387a5970327e4c0905ff943e1da7/class7_bezier/src/main/java/com/zinc/class7_bezier/widget/StickDotView.java")。

#### 效果图

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/12/1683f9e0f2802d4c~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

## 五、写在最后

贝塞尔曲线是小盆友自定义UI中最喜欢的一把利器，因为其线条的优美和控制的简单。希望这篇文章也能让你喜欢上她，并且挥舞起这把利器，做出更多的有个性的自定义组件。如果你从这篇文章有所收获，请给我个**赞**❤️，并关注我吧。文章中如有理解错误或是晦涩难懂的语句，请评论区留言，我们进行讨论共同进步。你的鼓励是我前进的最大动力。

高级UI系列的Github地址：请进入[传送门](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FzincPower%2FUI2018 "https://github.com/zincPower/UI2018")，如果喜欢的话给我一个star吧😄

  

作者：江澎涌  
链接：https://juejin.cn/post/6844903760007790600  
来源：稀土掘金  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。