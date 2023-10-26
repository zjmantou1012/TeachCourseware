---
author: zjmantou
title: 可能是最详细的Android图片压缩原理分析（一）—— Android图片压缩必备基础知识
time: 2023-10-26 周四
tags:
  - Android
  - 图片
---
本篇文章已授权微信公众号guolin_blog（郭霖）独家发布

## 前言：

        最近在研究图片压缩原理，看了大量资料，从上层尺寸压缩、质量压缩原理到下层的哈夫曼压缩，走成华大道，然后去二仙桥，全看了个遍，今天就来总结总结，做个技术分享，下面的内容可能会颠覆你对图片压缩的认知。

## 一、图片基础知识

**前言**

> 首先带着几个疑问来看这一小节：
> 
> 1、位深和色深有什么区别，他们是一个东西吗？
> 
> 2、为什么`Bitmap`不能直接保存，`Bitmap`和`PNG`、`JPG`到底是什么关系？
> 
> 3、图片占用的内存大小公式：**图片分辨率 * 每个像素点大小**，这种说法正确吗？
> 
> 4、为什么有时候同一个 app，app 内的同个界面上的同张图片，但在不同设备上所耗内存却不一样？
> 
> 5、同一张图片，在界面上显示的控件大小不同时，它的内存大小也会跟随着改变吗？

## （1）ARGB介绍

>     ARGB颜色模型：最常见的颜色模型，设备相关，四种通道，取值均为`[0，255]`，即转化成二进制位 `0000 0000 ~ 1111 1111`。

**A：Alpha (透明度) R：Red (红) G：Green (绿) B：Blue (蓝)**

## （2）Bitmap概念

>      `Bitmap`对象本质是一张图片的内容在手机内存中的表达形式。它将图片的内容看做是由存储数据的有限个像素点组成；每个像素点存储该像素点位置的`ARGB`值。每个像素点的`ARGB`值确定下来，这张图片的内容就相应地确定下来了。

>      关于Android中的两个类`Bitmap`和`BitmapFactory`，里面的重要函数请参考 [Android 图片缓存之 Bitmap 详解](https://juejin.cn/post/6844903442939412493#heading-8 "https://juejin.cn/post/6844903442939412493#heading-8")

## （3）色彩模式

**Android Bitmap常使用的色彩模式：**

     `Bitmap.Config`是`Bitmap`的一个枚举内部类，它表示的就是每个像素点对`ARGB`通道值的存储方案。取值有以下四种

> `ALPHA_8`：每个像素占8位（1个字节），存储透明度信息，没有颜色信息。

> `RGB_565`：没有透明度，R=5，G=6，B=5，，那么一个像素点占5+6+5=16位（2字节），能表示2^16种颜色。

> `ARGB_4444`：由4个4位组成，即A=4，R=4，G=4，B=4，那么一个像素点占4+4+4+4=16位 （2字节），能表示2^12种颜色。

> `ARGB_8888`：由4个8位组成，即A=8，R=8，G=8，B=8，那么一个像素点占8+8+8+8=32位（4字节），能表示2^24种颜色。

A是透明通道，代表灰度等级，严格意义上来说，不能算颜色。一个失去颜色的世界，就是黑白世界，达到极致，要么全白、要么全黑。

## （4）位深与色深

在windows上查看一张图片的信息会发现有位深度这个东西，但没看到有色深： ![在这里插入图片描述](file:///Users/mantou/.config/joplin-desktop/resources/900c9bced5384adba5ed2b51b37d57dc.webp)

**这里介绍一下位深与色深的概念：**

>     ① `色深`：顾名思义，就是"色彩的深度"，指是每一个像素点用多少bit来存储`ARGB`值，属于图片自身的一种属性。色深可以用来衡量一张图片的色彩处理能力（即色彩丰富程度）。典型的色深是**8-bit、16-bit、24-bit和32-bit**等。上述的`Bitmap.Config`参数的值指的就是色深。比如**ARGB_8888**方式的色深为**32**位，**RGB_565**方式的色深是**16**位。`色深是数字图像参数。`

> ②`位深度`是指在记录数字图像的颜色时，计算机实际上是用每个像素需要的二进制数值位数来表示的。当这些数据按照一定的编排方式被记录在计算机中，就构成了一个数字图像的计算机文件。`每一个像素在计算机中所使用的这种位数就是“位深度”，位深是物理硬件参数，主要用来存储。`
> 
>      举个例子：某张图片100像素*100像素 色深32位(ARGB_8888)，保存时位深度为24位，那么：
> 
> - 该图片在内存中所占大小为：100 * 100 * (32 / 8) Byte
> - 在文件中所占大小为 100 * 100 * ( 24/ 8 ) * 压缩率 Byte

> **拓展小知识：**
> 
>      24位颜色可称之为真彩色，色深度是24，它能组合成2的24次幂种颜色，即：16777216种颜色，超过了人眼能够分辨的颜色数量。

## （5）内存中Bitmap的大小

     网上很多文章都会介绍说，计算一张图片占用的内存大小公式：**分辨率 * 每个像素点的大小**，但事实真的如此吗？

>      我们都知道我们的手机屏幕有着一定的分辨率(如:**1920×1080**)，图像也有自己的像素(如拍摄图像的分辨率为**4032×3024**)。

>      如果将一张**1920×1080**的图片加载铺满**1920×1080**的屏幕上这就是最合适的了，此时显示效果最好。

>      如果将一张**4032×3024**的图像放到**1920×1080**的屏幕并不会得到更好的显示效果(和**1920×1080**的图像显示效果是一致的)，反而会浪费更多的内存，如果按ARGB_8888来显示的话，需要48MB的内存空间（4048*3036 *4 bytes），这么大的内存消耗极易引发OOM，后面我们会讲到针对[大图加载的内存优化](https://link.juejin.cn/?target= "https://link.juejin.cn/?target=")，在这里不过多介绍。

>      在 Android 原生的 `Bitmap` 操作中，图片来源是`res`内的不同资源目录时，图片被加载进内存时的分辨率会经过一层转换，所以，虽然最终图片大小的计算公式仍旧是分辨率*像素点大小，但此时的分辨率已不是图片本身的分辨率了。详细请看[字节跳动面试官：一张图片占据的内存大小是如何计算](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fweixin_43901866%2Farticle%2Fdetails%2F112169516%3Fspm%3D1035.2023.3001.6557%26utm_medium%3Ddistribute.pc_relevant_bbs_down.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-1.nonecase%26depth_1-utm_source%3Ddistribute.pc_relevant_bbs_down.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-1.nonecase "https://blog.csdn.net/weixin_43901866/article/details/112169516?spm=1035.2023.3001.6557&utm_medium=distribute.pc_relevant_bbs_down.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-1.nonecase&depth_1-utm_source=distribute.pc_relevant_bbs_down.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-1.nonecase")，规则如下：      `新分辨率 = 原图横向分辨率 * (设备的 dpi / 目录对应的 dpi ) * 原图纵向分辨率 * (设备的 dpi / 目录对应的 dpi )`。
> 
> - 当使用 `Glide` 时，如果有设置图片显示的控件，那么会自动按照控件的大小，降低图片的分辨率加载。图片来源是 `res` 的分辨率转换规则对它也无效。
> - 当使用 `fresco` 时，不管图片来源是哪里，即使是 `res`，图片占用的内存大小仍旧以原图的分辨率计算。
> - 其他图片的来源，如磁盘，文件，流等，均按照原图的分辨率来进行计算图片的内存大小。

**那么如何计算Bitmap占用的内存？**

`来看BitmapFactory.decodeResource()的源码：`

```
BitmapFactory.java
    public static Bitmap decodeResourceStream(Resources res, TypedValue value,InputStream is, Rect pad, Options opts) {
        if (opts == null) {
            opts = new Options();
        }
        if (opts.inDensity == 0 && value != null) {
            final int density = value.density;
            if (density == TypedValue.DENSITY_DEFAULT) {
                //inDensity默认为图片所在文件夹对应的密度
                opts.inDensity = DisplayMetrics.DENSITY_DEFAULT;
            } else if (density != TypedValue.DENSITY_NONE) {
                opts.inDensity = density;
            }
        }
        if (opts.inTargetDensity == 0 && res != null) {
            //inTargetDensity为当前系统密度。
            opts.inTargetDensity = res.getDisplayMetrics().densityDpi;
        }
        return decodeStream(is, pad, opts);
    }
    

    BitmapFactory.cpp 此处只列出主要代码。
    static jobject doDecode(JNIEnv* env, SkStreamRewindable* stream, jobject padding, jobject options) {
        //初始缩放系数
        float scale = 1.0f;
        if (env->GetBooleanField(options, gOptions_scaledFieldID)) {
            const int density = env->GetIntField(options, gOptions_densityFieldID);
            const int targetDensity = env->GetIntField(options, gOptions_targetDensityFieldID);
            const int screenDensity = env->GetIntField(options, gOptions_screenDensityFieldID);
            if (density != 0 && targetDensity != 0 && density != screenDensity) {
                //缩放系数是当前系数密度/图片所在文件夹对应的密度；
                scale = (float) targetDensity / density;
            }
        }
        //原始解码出来的Bitmap；
        SkBitmap decodingBitmap;
        if (decoder->decode(stream, &decodingBitmap, prefColorType, decodeMode)
                != SkImageDecoder::kSuccess) {
            return nullObjectReturn("decoder->decode returned false");
        }
        //原始解码出来的Bitmap的宽高；
        int scaledWidth = decodingBitmap.width();
        int scaledHeight = decodingBitmap.height();
        //要使用缩放系数进行缩放，缩放后的宽高；
        if (willScale && decodeMode != SkImageDecoder::kDecodeBounds_Mode) {
            scaledWidth = int(scaledWidth * scale + 0.5f);
            scaledHeight = int(scaledHeight * scale + 0.5f);
        }    
        //源码解释为因为历史原因；sx、sy基本等于scale。
        const float sx = scaledWidth / float(decodingBitmap.width());
        const float sy = scaledHeight / float(decodingBitmap.height());
        canvas.scale(sx, sy);
        canvas.drawARGB(0x00, 0x00, 0x00, 0x00);
        canvas.drawBitmap(decodingBitmap, 0.0f, 0.0f, &paint);
        // now create the java bitmap
        return GraphicsJNI::createBitmap(env, javaAllocator.getStorageObjAndReset(),
            bitmapCreateFlags, ninePatchChunk, ninePatchInsets, -1);
    }
```

**此处可以看出：加载一张本地资源图片，那么它占用的内存 = `width * height * nTargetDensity/inDensity * nTargetDensity/inDensity * 一个像素所占的内存`。**

## （6）小节总结

对小节问题给出答案：

> - 1、位深和色深看着其实差不多，都用24-bit和32-bit等来表示，而且一个图片的色深和位深有可能是一样（例如都是24bit），但一个是图片的参数，一个是用于存储的物理硬件参数。

> - 2、`Bitmap`是图片在内存中的表示形式，`GIF`、`JPEG`、`BMP`、`PNG`和`WebP`等格式图片是持久化存储后的图片， `jpg`、`png` 只是图片的容器。而当图片加载到内存中以显示的时候，应该将磁盘上压缩存储的图片内容完整地展开即为解压缩过程，所以`bitmap`很大，不能直接存储，在内存中很容易造成OOM。

> - 3、说法也对，但是不全对，没有说明场景，同时也忽略了一个影响项：`Density`，可以参考[解析Bitmap的density](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Faed7b7f010dc "https://www.jianshu.com/p/aed7b7f010dc")的讲解。公式应为： `width * height * nTargetDensity/inDensity * nTargetDensity/inDensity * 一个像素所占的内存`

> - 4、因为不同设备的分辨率不同，导致加载到内存中的大小也不一样

> - 5、热门的开源图片库，内部基本都会有一些图片的优化处理操作，当使用 `Glide` 时，会自动按照控件的大小，降低图片的分辨率加载，它的内存大小也会随着改变。

**一句话总结：** 这小节我们学会了**ARGB**是什么，**色彩模式**是什么有哪些，**位深与色深**的概念与关系，以及**Bitmap概念**和**内存中Bitmap的大小的计算公式**，掌握了这些基础我们就可以继续往下学习一下有关**图片压缩**的方法了。

## 二、Android中图片压缩的方法介绍

**前言：**

>      在 Android 中进行图片压缩是非常常见的开发场景，主要的压缩方法有两种：其一是质量压缩，其二是下采样压缩。

>      前者是在不改变图片尺寸的情况下，改变图片的存储体积，而后者则是降低图像尺寸，达到相同目的。

## （1）质量压缩

     在Android中，对图片进行质量压缩，通常我们的实现方式如下所示：

```
ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
//quality 为0～100，0表示最小体积，100表示最高质量，对应体积也是最大
bitmap.compress(Bitmap.CompressFormat.JPEG, quality , outputStream);
```

>      在上述代码中，我们选择的压缩格式是`CompressFormat.JPEG`，除此之外还有两个选择：
> 
> - 其一，`CompressFormat.PNG`， `PNG`格式是无损的，它无法再进行质量压缩，`quality` 这个参数就没有作用了，会被忽略，所以最后图片保存成的文件大小不会有变化；
> - 其二，`CompressFormat.WEBP` ，这个格式是 google 推出的图片格式，它会比 `JPEG` 更加省空间，经过实测大概可以优化 `30%` 左右。
> 
>      在某些应用场景需要`bitmap`转换成`ByteArrayOutputStream` ，需要根据你要压缩的图片格式来判断使用`CompressFormat.PNG`还是`Bitmap.CompressFormat.JPEG`， 这时候`quality` 为`100`。

**Android 质量压缩逻辑：**

     函数 `compress` 经过一连串的 java 层调用之后，最后来到了一个 native 函数，如下：

```
//Bitmap.cpp
static jboolean Bitmap_compress(JNIEnv* env, jobject clazz, jlong bitmapHandle,
                                jint format, jint quality,
                                jobject jstream, jbyteArray jstorage) {

    LocalScopedBitmap bitmap(bitmapHandle);
    SkImageEncoder::Type fm;

    switch (format) {
    case kJPEG_JavaEncodeFormat:
        fm = SkImageEncoder::kJPEG_Type;
        break;
    case kPNG_JavaEncodeFormat:
        fm = SkImageEncoder::kPNG_Type;
        break;
    case kWEBP_JavaEncodeFormat:
        fm = SkImageEncoder::kWEBP_Type;
        break;
    default:
        return JNI_FALSE;
    }

    if (!bitmap.valid()) {
        return JNI_FALSE;
    }

    bool success = false;

    std::unique_ptr<SkWStream> strm(CreateJavaOutputStreamAdaptor(env, jstream, jstorage));
    if (!strm.get()) {
        return JNI_FALSE;
    }

    std::unique_ptr<SkImageEncoder> encoder(SkImageEncoder::Create(fm));
    if (encoder.get()) {
        SkBitmap skbitmap;
        bitmap->getSkBitmap(&skbitmap);
        success = encoder->encodeStream(strm.get(), skbitmap, quality);
    }
    return success ? JNI_TRUE : JNI_FALSE;
}
```

可以看到最后调用了函数 encoder->encodeStream(....) 编码保存本地。该函数是调用 `skia 引擎`来对图片进行编码压缩，对`skia` 的介绍将在后文讲解 [底层哈夫曼压缩](https://juejin.cn/post/7036719089804394532 "https://juejin.cn/post/7036719089804394532")时展开

## （2）尺寸压缩

### 1、邻近采样（Nearest Neighbour Resampling）

```
BitmapFactory.Options options = new BitmapFactory.Options();
//或者 inDensity 搭配 inTargetDensity 使用，算法和 inSampleSize 一样
options.inSampleSize = 2; //设置图片的缩放比例(宽和高) , google推荐用2的倍数：
Bitmap bitmap = BitmapFactory.decodeFile("xxx.png");
Bitmap compress = BitmapFactory.decodeFile("xxx.png", options);
```

     在这里着重讲一下这个`inSampleSize`。从字面上理解，它的含义是: "设置取样大小"。它的作用是：设置`inSampleSize`的值(`int`类型)后，假如设为`4`，则宽和高都为原来的`1/4`，宽高都减少了，自然内存也降低了。

>      参考[Google官方文档](https://link.juejin.cn/?target=http%3A%2F%2Fdeveloper.android.com%2Fintl%2Fzh-cn%2Ftraining%2Fdisplaying-bitmaps%2Fload-bitmap.html%23load-bitmap "http://developer.android.com/intl/zh-cn/training/displaying-bitmaps/load-bitmap.html#load-bitmap")的解释，我们从中可以看到 x（x 为 2 的倍数）个像素最后对应一个像素，由于采样率设置为 1/2，所以是两个像素生成一个像素。

>      **邻近采样**的方式比较粗暴，直接选择其中的一个像素作为生成像素，另一个像素直接抛弃，这样就造成了图片变成了纯绿色，也就是红色像素被抛弃。

**邻近采样**采用的算法叫做**邻近点插值算法**。

### 2、双线性采样（Bilinear Resampling）

双线性采样（Bilinear Resampling）在 Android 中的使用方式一般有两种：

```
Bitmap bitmap = BitmapFactory.decodeFile("xxx.png");
Bitmap compress = Bitmap.createScaledBitmap(bitmap, bitmap.getWidth()/2, bitmap.getHeight()/2, true);
或者直接使用 matrix 进行缩放

Bitmap bitmap = BitmapFactory.decodeFile("xxx.png");
Matrix matrix = new Matrix();
matrix.setScale(0.5f, 0.5f);
bm = Bitmap.createBitmap(bitmap, 0, 0, bit.getWidth(), bit.getHeight(), matrix, true);
```

     看源码可以知道 `createScaledBitmap` 函数最终也是使用第二种方式的 `matrix` 进行缩放，**双线性采样**使用的是**双线性內插值算法**，这个算法不像**邻近点插值**算法一样，直接粗暴的选择一个像素，而是参考了源像素相应位置周围 2x2 个点的值，根据相对位置取对应的权重，经过计算之后得到目标图像。

     **双线性内插值**算法在图像的缩放处理中具有抗锯齿功能, 是最简单和常见的图像缩放算法，当对相邻 2x2 个像素点采用双线性內插值算法时，所得表面在邻域处是吻合的，但斜率不吻合，并且双线性内插值算法的平滑作用可能使得图像的细节产生退化，这种现象在上采样时尤其明显。

>      **双线性采样**对比**邻近采样**的优势在于：
> 
> - 它的系数可以是小数，而不一定是整数，在某些压缩限制下，效果尤为明显
> - 处理文字比较多的图片在展示效果上的差别，双线性采样效果要更好

还有**双三次采样**和**Lanczos**采样等，具体分析可以参考 [Android 中图片压缩分析（下）](https://link.juejin.cn/?target=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1006352 "https://cloud.tencent.com/developer/article/1006352")这篇QQ音乐大佬的分享。

## （3）小节总结

>      在 Android 中，前两种采样方法根据实际情况去选择即可，如果对时间要求不高，倾向于使用**双线性采样**去缩放图片。如果对图片质量要求很高，**双线性采样**也已经无法满足要求，则可以考虑引入另外几种算法去处理图片，但是同时需要注意的是后面两种算法使用的都是卷积核去计算生成像素，计算量会相对比较大，**Lanczos**的计算量则是最大，在**实际开发过程**中根据需求进行算法的选择即可，往往我们是**尺寸压缩**和**质量压缩**搭配来使用。

>      下面我们要进入到实战中，**参考一个仿微信朋友圈压缩策略的Android图片压缩工具**——[Luban](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FCurzibn%2FLuban "https://github.com/Curzibn/Luban")，进入我们的下一章节[鲁班压缩算法解析](https://juejin.cn/post/7036716428174557221 "https://juejin.cn/post/7036716428174557221")。