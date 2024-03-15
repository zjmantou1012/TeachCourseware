---
author: zjmantou
title: 可能是最详细的Android图片压缩原理分析（二）—— 鲁班压缩算法解析
time: 2023-10-26 周四
tags:
  - Android
  - 图片
---
本篇文章已授权微信公众号guolin_blog（郭霖）独家发布

## 三、鲁班压缩算法解析

**前言**

>         通过前两章，我们了解了一些关于[图片压缩的基础知识](https://juejin.cn/post/7036714008644157447 "https://juejin.cn/post/7036714008644157447")，这篇文章我们主要讲解一下鲁班压缩的算法逻辑，很多博客都是从Github上将人家的介绍直接拷贝过来，根本没有什么讲解，就是告诉你一些方法如何调用。

## （1）鲁班压缩的背景

[鲁班压缩](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FCurzibn%2FLuban "https://github.com/Curzibn/Luban") —— Android图片压缩工具，仿微信朋友圈压缩策略。

>         目前做**App**开发总绕不开图片这个元素。但是随着手机拍照分辨率的提升，**图片的压缩**成为一个很**重要**的**问题**，随便一张图片都是好几M，甚至**几十M**，这样的照片加载到**app**，可想而知，随便加载几张图片，手机内存就不够用了，自然而然就造成了`OOM` ，所以，Android的**图片压缩**异常**重要**。

>         单纯对图片进行裁切，压缩已经有很多文章介绍。但是**裁切成多少**，**压缩成多少**却很难控制好，裁切过头图片太小，质量压缩过头则显示效果太差。于是自然想到**App巨头——微信**会是怎么处理，`Luban`（鲁班）就是通过在**微信朋友圈发送近100张不同分辨率图片**，**对比原图**与微信压缩后的图片**逆向推算**出来的压缩算法。

## （2）效果与对比

        因为是逆向推算，效果还没法跟微信一模一样，但是已经很接近微信朋友圈压缩后的效果，具体看以下对比！

|内容|原图|Luban|Wechat|
|---|---|---|---|
|截屏 720P|720*1280，390k|720*1280，87k|720*1280，56k|
|截屏 1080P|1080*1920，2.21M|1080*1920，104k|1080*1920，112k|
|拍照 13M(4:3)|3096*4128，3.12M|1548*2064，141k|1548*2064，147k|
|拍照 9.6M(16:9)|4128*2322，4.64M|1032*581，97k|1032*581，74k|
|滚动截屏|1080*6433，1.56M|1080*6433，351k|1080*6433，482k|

## （3）Luban算法解析

### 1、微信的算法解析

- 第一步进行采样率压缩；
- 第二步进行宽高的等比例压缩（微信对原图和缩略图限制了最大长宽或者最小长宽）；
- 第三步就是对图片的质量进行压缩（一般75或者70）；
- 第四部就是采用webP的格式。

经过这四部的处理，基本上和微信朋友圈的效果一致，包括文件大小和显示效果

### 2、Luban的算法解析

`Luban`压缩目前的步骤只占了微信算法中的第二与第三步，算法逻辑如下：

**1. 判断图片比例值，是否处于以下区间内；**

> - [1, 0.5625) 即图片处于 [1:1 ~ 9:16) 比例范围内
> - [0.5625, 0.5) 即图片处于 [9:16 ~ 1:2) 比例范围内
> - [0.5, 0) 即图片处于 [1:2 ~ 1:∞) 比例范围内

        简单解释一下：获取图片的比例系数，如果在区间 `[1, 0.5625)` 中即图片处于 `[1:1 ~ 9:16)`比例范围内，比例以此类推，如果这个系数小于`0.5`，那么就给它放到 `[1:2 ~ 1:∞)` 比例范围内。

**2. 判断图片最长边是否过边界值；**

> - [1, 0.5625) 边界值为：1664 * n（n=1）, 4990 * n（n=2）, 1280 * pow(2, n-1)（n≥3）
> - [0.5625, 0.5) 边界值为：1280 * pow(2, n-1)（n≥1）
> - [0.5, 0) 边界值为：1280 * pow(2, n-1)（n≥1）

        步骤二：上去一看一脸懵，1664是什么，n是什么，pow又是什么。。。这写的估计只有作者自己能看懂了。其实就是判断图片最长边是否过边界值，此边界值是模仿微信的一个经验值，就是说`1664、4990`都是经验值，**模仿微信的策略**。

        至于`n`，是返回的是 `options.inSampleSize`的值，就是采样压缩的系数，是`int`型，Google建议是`2`的倍数，所以为了配合这个建议，代码中出现了`小于10240`返回的是`4`这种操作。最后说一下`pow`，其实是 `（长边/1280）`, 这个`1280`也是个经验值，逆向推出来的，解释到这里逻辑也清晰了。真是坑啊啊，哈哈哈

**3. 计算压缩图片实际边长值，以第2步计算结果为准，超过某个边界值则：**

> - width / pow(2, n-1)，
> - height/ pow(2, n-1)

        步骤三：这个感觉没什么用，还是计算压缩图片实际边长值，人家也说了，以**第2步计算结果为准**，其实就是晃你的，乍一看 ，这么多步骤，哈哈哈哈，唬你呢！

**4. 计算压缩图片的实际文件大小，以第2、3步结果为准，图片比例越大则文件越大。**

> size = (newW * newH) / (width * height) * m；
> 
> - [1, 0.5625) 则 width & height 对应 1664，4990，1280 * n（n≥3），m 对应 150，300，300；
> - [0.5625, 0.5) 则 width = 1440，height = 2560, m = 200；
> - [0.5, 0) 则 width = 1280，height = 1280 / scale，m = 500；注：scale为比例值

        步骤四：这个感觉也没什么用，这个`m`应该是压缩比。但整个过程就是验证一下压缩完之后，`size`的大小，是否超过了你的预期，如果超过了你的预期，将进行重复压缩。

**5. 判断第4步的size是否过小**

> - [1, 0.5625) 则最小 size 对应 60，60，100
> - [0.5625, 0.5) 则最小 size 都为 100
> - [0.5, 0) 则最小 size 都为 100

        步骤五：这一步也没啥用，也是为了后面循环压缩使用。 这个`size`就是上面计算出来的，最小 size 对应的值公式为：`size = (newW * newH) / (width * height) * m`，对应的三个值，就是上面根据图片的比例分成的三组，然后计算出来的。

**6. 将前面求到的值压缩图片 width, height, size 传入压缩流程，压缩图片直到满足以上数值**

        最后一步也没啥用，看字就知道是为了循环压缩，或许是微信也这样做？既然你已经有了预期，为什么不根据预期直接一步到位呢？但是`裁剪的系数`和`压缩的系数`怎么调整会达到最优一个效果，我的项目中已经对此功能进行了增加，目前还在内测，没有开源，后期稳定后会开源给大家使用。

### 3、将算法带入到开源代码中

咱们直接看算法所在类 [Engine.java](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FCurzibn%2FLuban%2Fblob%2Fmaster%2Flibrary%2Fsrc%2Fmain%2Fjava%2Ftop%2Fzibin%2Fluban%2FEngine.java "https://github.com/Curzibn/Luban/blob/master/library/src/main/java/top/zibin/luban/Engine.java")

```
// 计算采样压缩的值，也就是模仿微信的经验值，核心内容
private int computeSize() {
// 补齐宽度和长度
    srcWidth = srcWidth % 2 == 1 ? srcWidth + 1 : srcWidth;
    srcHeight = srcHeight % 2 == 1 ? srcHeight + 1 : srcHeight;

// 获取长边和短边
    int longSide = Math.max(srcWidth, srcHeight);
    int shortSide = Math.min(srcWidth, srcHeight);

// 获取图片的比例系数，如果在区间[1, 0.5625) 中即图片处于 [1:1 ~ 9:16) 比例
    float scale = ((float) shortSide / longSide);
// 开始判断图片处于那种比例中，就是上面所说的第一个步骤
    if (scale <= 1 && scale > 0.5625) {
// 判断图片最长边是否过边界值，此边界值是模仿微信的一个经验值，就是上面所说的第二个步骤
      if (longSide < 1664) {
// 返回的是 options.inSampleSize的值，就是采样压缩的系数，是int型，Google建议是2的倍数
        return 1;
      } else if (longSide < 4990) {
        return 2;
// 这个10240上面的逻辑没有提到，也是经验值，不用去管它，你可以随意调整
      } else if (longSide > 4990 && longSide < 10240) {
        return 4;
      } else {
        return longSide / 1280 == 0 ? 1 : longSide / 1280;
      }
// 这些判断都是逆向推导的经验值，也可以说是一种策略
    } else if (scale <= 0.5625 && scale > 0.5) {
      return longSide / 1280 == 0 ? 1 : longSide / 1280;
    } else {
    // 此时图片的比例是一个长图，采用策略向上取整
      return (int) Math.ceil(longSide / (1280.0 / scale));
    }
  }
  

// 图片旋转方法
private Bitmap rotatingImage(Bitmap bitmap, int angle) {
    Matrix matrix = new Matrix();
// 将传入的bitmap 进行角度旋转
    matrix.postRotate(angle);
// 返回一个新的bitmap
    return Bitmap.createBitmap(bitmap, 0, 0, bitmap.getWidth(), bitmap.getHeight(), matrix, true);
  }

// 压缩方法，返回一个File
File compress() throws IOException {
// 创建一个option对象
    BitmapFactory.Options options = new BitmapFactory.Options();
// 获取采样压缩的值
    options.inSampleSize = computeSize();
// 把图片进行采样压缩后放入一个bitmap， 参数1是bitmap图片的格式，前面获取的
    Bitmap tagBitmap = BitmapFactory.decodeStream(srcImg.open(), null, options);
// 创建一个输出流的对象
    ByteArrayOutputStream stream = new ByteArrayOutputStream();
// 判断是否是JPG图片
    if (Checker.SINGLE.isJPG(srcImg.open())) {
// Checker.SINGLE.getOrientation这个方法是检测图片是否被旋转过，对图片进行矫正
      tagBitmap = rotatingImage(tagBitmap, Checker.SINGLE.getOrientation(srcImg.open()));
    }
// 对图片进行质量压缩，参数1：通过是否有透明通道来判断是PNG格式还是JPG格式，
// 参数2：压缩质量固定为60，参数3：压缩完后将bitmap写入到字节流中
    tagBitmap.compress(focusAlpha ? Bitmap.CompressFormat.PNG : Bitmap.CompressFormat.JPEG, 60, stream);
// bitmap用完回收掉
    tagBitmap.recycle();
// 将图片流写入到File中，然后刷新缓冲区，关闭文件流和Byte流
    FileOutputStream fos = new FileOutputStream(tagImg);
    fos.write(stream.toByteArray());
    fos.flush();
    fos.close();
    stream.close();
    return tagImg;
  }
```

整个讲解已经在代码里已经做了注释，要结合前面逻辑看，就明白了。

## （4）Luban原框架问题分析

### 1、原框架问题分析

> - 解码前没有对内存做出预判
> - 质量压缩写死 60
> - 没有提供图片输出格式选择
> - 不支持多文件合理并行压缩，输出顺序和压缩顺序不能保证一致
> - 检测文件格式和图像的角度多次重复创建InputStream,增加不必要开销，增加OOM风险
> - 可能出现内存泄漏，需要自己合理处理生命周期
> - 图片要是有大小限制，只能进行重复压缩
> - 原框架用的还是RxJava1.0

### 2、技术改造方案

> - 解码前利用获取的图片宽高对内存占用做出计算，超出内存的使用RGB-565尝试解码
> - 针对质量压缩的时候，提供传入质量系数的接口
> - 对图片输出支持多种格式，不局限于File
> - 利用协程来实现异步压缩和并行压缩任务，可以在合适时机取消协程来终止任务
> - 参考Glide对字节数组的复用，以及InputStream的mark()、reset()来优化重复打开开销
> - 利用LiveData来实现监听，自动注销监听。
> - 压缩前计算好大小，逆向推导出尺寸压缩系数和质量压缩系数
> - 现在已经出了RxJava3和协程，但大多数项目中已经有了线程池，要利用项目中的线程池，而不是导入一个三方库就建一个线程池而造成资源浪费

## （5）小节总结

### 1、结语

        `Luban`压缩当初出来的时候号称 `"可能是最接近微信朋友圈的图片压缩算法"` ，但这个库已经三四年没有维护了，随着产品的迭代微信已经也不是当初的那个微信了,`Luban`压缩的库也要进行更新了。所以为了适应现在的项目，我之后会根据上面的技术改造方案对图片压缩出一个船新版本的库，更为强大。

### 2、再续前缘

        `Luban`还有一个`turbo`分支，这个分支主要是为了兼容`Android 7.0`以前的系统版本，导入`libjpeg-turbo`的`jni`版本。

        `libjpeg-turbo`是一个C语音编写的高效JPEG图像处理库，Android系统在7.0版本之前内部使用的是`libjpeg非turbo版`，并且为了性能关闭了`Huffman编码`。在7.0之后的系统内部使用了`libjpeg-turbo`库并且启用`Huffman编码`。

        那么什么是`Huffman编码`呢？前面提到的`skio`引擎又是什么东西呢？让我们一起进入新的篇章：[底层哈夫曼压缩讲解](https://juejin.cn/post/7036719089804394532 "https://juejin.cn/post/7036719089804394532")