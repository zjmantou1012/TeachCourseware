---
author: zjmantou
title: Android中图片压缩分析（下）
time: 2023-10-26 周四
tags:
  - Android
  - 图片
---
> 作者： shawnzhao

[上篇](https://www.qcloud.com/community/article/427566?from_column=20421&from=20421 "https://www.qcloud.com/community/article/427566?from_column=20421&from=20421")我们详细介绍了图片质量压缩的相关内容和算法，接下来的下篇给大家介绍一下图片的尺寸压缩和常用的几种尺寸压缩算法。

针对图片尺寸的修改其实就是一个图像重新采样的过程，放大图像称为上采样（upsamping），缩小图像称为下采样（downsampling），这里我们重点讨论下采样。

在 Android 中图片重采样提供了两种方法，一种叫做邻近采样（Nearest Neighbour Resampling），另一种叫做双线性采样（Bilinear Resampling）。

除了 Android 中这两种常用的重采样方法之外，还有另外比较常见的两种：双立方／双三次采样（Bicubic Resampling） 和 Lanczos Resampling。除此之外，还有一些其他个人或机构发明的算法 Hermite Resampling，Bell Resampling，Mitchell Resampling。我们这里着重介绍前面提到的四种采样方法。

## 二、邻近采样（Nearest Neighbour Resampling）

Nearest Neighbour Resampling（邻近采样），是 Android 中常用的压缩方法之一，我们先来看看在 Android 中使用邻近采样的示例代码：

```

BitmapFactory.Options options = new BitmapFactory.Options();
//或者 inDensity 搭配 inTargetDensity 使用，算法和 inSampleSize 一样
options.inSampleSize = 2;
Bitmap bitmap = BitmapFactory.decodeFile("/sdcard/test.png");
Bitmap compress = BitmapFactory.decodeFile("/sdcard/test.png", options);
```

来看看邻近采样的图片效果：
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310261554712.png)


图是每个像素红绿相间的图片，可以看到处理之后的图片已经完全变成了绿色，接着我们来看看 inSampleSzie 的官方描述：

> If set to a value > 1, requests the decoder to subsample the original image, returning a smaller image to save memory. The sample size is the number of pixels in either dimension that correspond to a single pixel in the decoded bitmap. For example, inSampleSize == 4 returns an image that is 1/4 the width/height of the original, and 1/16 the number of pixels. Any value <= 1 is treated the same as 1. Note: the decoder uses a final value based on powers of 2, any other value will be rounded down to the nearest power of 2.

从官方的解释中我们可以看到 x（x 为 2 的倍数）个像素最后对应一个像素，由于采样率设置为 1/2，所以是两个像素生成一个像素。邻近采样的方式比较粗暴，直接选择其中的一个像素作为生成像素，另一个像素直接抛弃，这样就造成了图片变成了纯绿色，也就是红色像素被抛弃。

邻近采样采用的算法叫做邻近点插值算法。

## 三、双线性采样（Bilinear Resampling）

双线性采样（Bilinear Resampling）在 Android 中的使用方式一般有两种：

```Java
Bitmap bitmap = BitmapFactory.decodeFile("/sdcard/test.png");
Bitmap compress = Bitmap.createScaledBitmap(bitmap, bitmap.getWidth()/2, bitmap.getHeight()/2, true);
或者直接使用 matrix 进行缩放

Bitmap bitmap = BitmapFactory.decodeFile("/sdcard/test.png");
Matrix matrix = new Matrix();
matrix.setScale(0.5f, 0.5f);
bm = Bitmap.createBitmap(bitmap, 0, 0, bit.getWidth(), bit.getHeight(), matrix, true);
```

看源码可以知道 createScaledBitmap 函数最终也是使用第二种方式的 matrix 进行缩放，我们来看看双线性采样的表现：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310261554318.png)


可以看到处理之后的图片不是像邻近采样一样纯粹的一种颜色，而是两种颜色的混合。双线性采样使用的是双线性內插值算法，这个算法不像邻近点插值算法一样，直接粗暴的选择一个像素，而是参考了源像素相应位置周围 2x2 个点的值，根据相对位置取对应的权重，经过计算之后得到目标图像。

双线性内插值算法在图像的缩放处理中具有抗锯齿功能, 是最简单和常见的图像缩放算法，当对相邻 2x2 个像素点采用双线性內插值算法时，所得表面在邻域处是吻合的，但斜率不吻合，并且双线性内插值算法的平滑作用可能使得图像的细节产生退化，这种现象在上采样时尤其明显。

## 四、邻近采样和双线性采样对比

我们这里来对比一下这两种 Android 中经常用到的图片尺寸压缩方法。

邻近采样的方式是最快的，因为它直接选择其中一个像素作为生成像素，但是生成的图片可能会相对比较失真，产生比较明显的锯齿，最具有代表性的就是处理文字比较多的图片在展示效果上的差别，对比：

原图：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310261555561.png)


邻近采样：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310261555376.png)


双线性采样：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310261555924.png)


这个对比就非常直观了，邻近采样字的显示失真对比双线性采样来说要严重很多。

## 五、双立方／双三次采样（Bicubic Resampling）

双立方／双三次采样使用的是双立方／双三次插值算法。邻近点插值算法的目标像素值由源图上单个像素决定，双线性內插值算法由源像素某点周围 2x2 个像素点按一定权重获得，而双立方／双三次插值算法更进一步参考了源像素某点周围 4x4 个像素。

这个算法在 Android 中并没有原生支持，如果需要使用，可以通过手动编写算法或者引用第三方算法库，幸运的是这个算法在 ffmpeg 中已经给到了支持，具体的实现在 libswscale/swscale.c 文件中：FFmpeg Scaler Documentation。

双立方/双三次插值算法经常用于图像或者视频的缩放，它能比双线性内插值算法保留更好的细节质量。我们看看这个算法的实际表现和与双线性内插值算法的下采样对比：

原图：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310261555589.png)


双三次采样：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310261555087.png)


双线性采样：
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310261556410.png)


就下采样来说，两者表现很相近，肉眼可见的差距不大，接下来比较一下这两种算法的上采样实际表现：

原图：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310261556253.png)


双三次采样：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310261556064.png)


双线性采样：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310261556301.png)


这两种算法的上采样结果我们还是可以看见较为明显的差距，双立方/双三次采样的锯齿是要小一些。

双立方／双三次插值算法在平时的软件中是很常用的一种[图片处理](https://cloud.tencent.com/product/ip?from_column=20065&from=20065 "https://cloud.tencent.com/product/ip?from_column=20065&from=20065")算法，但是这个算法有一个缺点就是计算量会相对比较大，是前三种算法中计算量最大的，软件 photoshop 中的图片缩放功能使用的就是这个算法。

## 六、Lanczos Resampling

Lanczos 采样和 Lanczos 过滤是 Lanczos 算法的两种常见应用，它可以用作低通滤波器或者用于平滑地在采样之间插入数字信号，Lanczos 采样一般用来增加数字信号的采样率，或者间隔采样来降低采样率。

Lanczos 采样使用的 Lanczos 算法也可以用来作为图片的缩放，Lanczos 算法和双三次插值算法都是使用卷积核来通过输入像素计算输出像素，只不过在算法表现上稍有不同。关于卷积核的介绍，这里给一张简单的图片帮助大家理解：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310261557747.png)


Lanczos 从算法角度讲理论上会比双三次／双立方插值算法更好一点，先来看看它和双三次／双立方采样的图片下采样对比：

原图：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310261555589.png)

Lanczos 采样：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310261558570.png)


双三次采样：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202310261558948.png)


基本看不出差别，然后是这两种算法的上采样对比：

原图：

![](file:///Users/mantou/.config/joplin-desktop/resources/b09e726f5e4e4037834778c3656eca72.jpeg)

Lanczos 采样：

![](file:///Users/mantou/.config/joplin-desktop/resources/5df7b1eafe994bb2bbfe00329a196c68.jpeg)

双三次采样：

![](file:///Users/mantou/.config/joplin-desktop/resources/bf8946aa1aa2407ebd1eb09f644a5143.jpeg)

这两种算法的上下采样结果从肉眼上看差距很小，但是从理论上来说 Lanczos 算法处理出来的图片应该是更加平滑少锯齿的。

同样的，Lanczos 算法在 ffmpeg 的 libswscale/swscale.c 中也有实现。其实不光 Lanczos 和上面的三种算法，ffmpeg 还提供了其他的图像重采样方法，诸如 area averaging、Gaussian 等等，通过编译好的 ffmpeg 库调用这些算法处理图片的命令如下：

```
ffmpeg -s 600x500 -i input.jpg -s 300x250 -sws_flags lanczos lanczos.jpg
```

-sws_flags 参数根据采样算法可以选择 bilinear/bicubic/lanczos 等等。

## 七、四种算法对二值化图片的处理表现

这四种图片重采样算法在处理二值化图片上面的表现差异较大，我们先看看下采样的对比：

原图：

![](file:///Users/mantou/.config/joplin-desktop/resources/39b0159fb7b444dcb70dfa23e2ab1c3d.png)

邻近采样：

![](file:///Users/mantou/.config/joplin-desktop/resources/a6a3c1c107c340929163c1f6792a847d.png)

双线性采样：

![](file:///Users/mantou/.config/joplin-desktop/resources/1558fbe36f5a48089abaa16773aba824.png)

双三次采样：

![](file:///Users/mantou/.config/joplin-desktop/resources/eeda53ed3a694e7482704dcda01ad58b.png)

Lanczos 采样：

![](file:///Users/mantou/.config/joplin-desktop/resources/d77f80bb569f4c45988cb5886e2a3c41.png)

下采样的对比一目了然，从上到下的图像表现效果逐渐变优，Lanczos 算法处理后的图像质量属于最优，接着我们看看这四种算法的上采样对比：

原图：

![](file:///Users/mantou/.config/joplin-desktop/resources/9beff2d365654a57893db71aca1f7ee7.png)

邻近采样：

![](file:///Users/mantou/.config/joplin-desktop/resources/091e4c25d2974652913d5f15003c8aa2.png)

双线性采样：

![](file:///Users/mantou/.config/joplin-desktop/resources/69c2016c454845e8a9d1ff5870210f67.png)

双三次采样：

![](file:///Users/mantou/.config/joplin-desktop/resources/e9627ace18d94e68a1cf74d207111155.png)

Lanczos 采样：

![](file:///Users/mantou/.config/joplin-desktop/resources/c5c3dba2d4634e768879c6604fce60c8.png)

从图像质量上来看，和下采样结果一致，邻近采样效果较差，依次往下效果变优，Lanczos 效果最优。

## 八、总结

上面主要介绍了常见的四种图像重采样算法，在 Android 中，前两种采样方法根据实际情况去选择即可，如果对时间要求不高，倾向于使用双线性采样去缩放图片。如果对图片质量要求很高，双线性采样也已经无法满足要求，则可以考虑引入另外几种算法去处理图片，但是同时需要注意的是后面两种算法使用的都是卷积核去计算生成像素，计算量会相对比较大，Lanczos 的计算量则是最大，在实际开发过程中根据需求进行算法的选择即可。

本文仅代表作者观点