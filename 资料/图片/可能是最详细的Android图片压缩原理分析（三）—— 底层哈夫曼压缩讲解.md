---
author: zjmantou
title: 可能是最详细的Android图片压缩原理分析（三）—— 底层哈夫曼压缩讲解
time: 2023-10-26 周四
tags:
  - Android
  - 图片
---
本篇文章已授权微信公众号guolin_blog（郭霖）独家发布

## 四、底层哈夫曼压缩讲解

**前言**

>      在前面的 [Android图片压缩必备基础知识](https://juejin.cn/post/7036714008644157447 "https://juejin.cn/post/7036714008644157447") 中，提到的`Skia`是Android的重要组成部分。在[鲁班压缩算法解析](https://juejin.cn/post/7036716428174557221 "https://juejin.cn/post/7036716428174557221")中提到哈夫曼压缩，那么他们之间到底是什么关系呢？

## （1）Android Skia 图像引擎

**Android Skia 图像引擎**

>      `Skia` 是一个2D向量图形处理函数库， 2005年被Google收购后并自己维护的 c++ 实现的图像引擎，实现了各种图像处理功能，并且广泛地应用于谷歌自己和其它公司的产品中（如：Chrome、Firefox、 Android等），基于它可以很方便为操作系统、浏览器等开发图像处理功能。

>      `Skia` 在 Android 中提供了基本的画图和简单的编解码功能，可以挂接其他的第三方编码解码库或者硬件编解码库，例如 **libpng 和 libjpeg ，libgif** 等等。因此，这个函数调用`bitmap.compress(Bitmap.CompressFormat.JPEG...)`，实际会调用 **libjpeg.so**动态库进行编码压缩。

>      最终 Android 编码保存图片的逻辑是 **Java 层函数→Native 函数→Skia函数→对应第三库函数**（例如 libjpeg）。所以`skia`就像一个 **胶水层**，用来链接各种第三方编解码库，不过 Android 也会对这些库做一些修改，比如修改内存管理的方式等等。

>      Android 在之前从某种程度来说使用的算是 **libjpeg** 的功能阉割版，压缩图片默认使用的是 `standard huffman`，而不是 `optimized huffman`，也就是说使用的是`默认的哈夫曼表`，并没有根据实际图片去计算相对应的哈夫曼表，Google 在初期考虑到手机的性能瓶颈，计算图片权重这个阶段非常占用 CPU 资源的同时也非常耗时，因为此时需要计算图片所有像素 argb 的权重，这也是 Android 的图片压缩率对比 iOS 来说差了一些的原因之一。

## （2）图像压缩与Huffman算法

     这里简单介绍一下`哈夫曼算法`，`哈夫曼算法`是在多媒体处理里常用的算法之一。比如一个文件中可能会出现五个值 `a,b,c,d,e`，它们用二进制表达是：

> a. 1010 b. 1011 c. 1100 d. 1101 e. 1110

     我们可以看到，最前面的一位数字是 1，其实是浪费掉了，在定长算法下最优的表达式为：

> a. 010 b. 011 c. 100 d. 101 e. 110

     这样我们就能做到节省一位的损耗，那哈夫曼算法比起定长算法改进的地方在哪里呢？在哈夫曼算法中我们可以给信息赋予权重，即为信息加权重，**假设 a 占据了 60%，b 占据了 20%， c 占据了 20%，d,e 都是 0%**：

> a:010 (60%) b:011 (20%) c:100 (20%) d:101 (0%) e:110 (0%)

     在这种情况下，我们可以使用哈夫曼树算法再次优化为：

> a:1 b:01 c:00

     所以思路当然就是出现频率高的字母使用短码，对出现频率低的使用长码，不出现的直接就去掉，最后 abcde 的哈夫曼编码就对应：`1 01 00`

     定长编码下的`abcde`：`010 011 100 101 110`， 使用 哈夫曼树 加权重后的 编码则为 `1 01 00`，这就是哈夫曼算法的整体思路（关于算法的详细介绍可以参考[哈夫曼树及编码讲解及例题](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fweixin_41738030%2Farticle%2Fdetails%2F89402695 "https://blog.csdn.net/weixin_41738030/article/details/89402695")）。

     所以这个算法一个很重要的思路是必须知道每一个元素出现的权重，如果我们能够知道每一个元素的权重，那么就能够根据权重动态生成一个最优的哈夫曼表。

     但是怎么去获取每一个元素，对于图片就是每一个像素中 argb 的权重呢，只能去循环整个图片的像素信息，这无疑是非常消耗性能的，所以早期 android 就使用了默认的哈夫曼表进行图片压缩。

## （3）libjpeg 与 optimize_coding

     `libjpeg`在压缩图像时，有一个参数叫 `optimize_coding`，关于这个参数，`libjpeg.doc` 有如下解释：

> TRUE causes the compressor to compute optimal Huffman coding tables for the image. This requires an extra pass over the data and therefore costs a good deal of space and time. The default is FALSE, which tells the compressor to use the supplied or default Huffman tables. In most cases optimal tables save only a few percent of file size compared to the default tables. Note that when this is TRUE, you need not supply Huffman tables at all, and any you do supply will be overwritten.

     由上可知，如果设置 `optimize_coding` 为`TRUE`，将会使得压缩图像过程中，会先基于图像数据计算哈弗曼表。由于这个计算会显著消耗空间和时间，默认值被设置为`FALSE`。

     那么 `optimize_coding` 参数的影响究竟会有多大呢？Skia 的官方人员经过实际测试，分别设置 `optimize_coding=TRUE` 和 `FALSE` 进行压缩，发现 `FALSE` 时的图片大小大约是 `TRUE` 时的 **2倍+**。换言之就是相同文件体积的图片，不使用哈夫曼编码图片质量会比使用哈夫曼低 **2倍+**。

     从 Android 7.0 版本开始，`optimize_code` 标示已经设置为了 `TRUE`，也就是默认使用图像生成哈夫曼表，而不是使用默认哈夫曼表。

     本章内容借鉴了[Android 中图片压缩分析（上)](https://link.juejin.cn/?target=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1006307 "https://cloud.tencent.com/developer/article/1006307")中的内容，自认为不能比他写的更好，感谢[QQ音乐技术团队](https://link.juejin.cn/?target=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Fuser%2F1027752 "https://cloud.tencent.com/developer/user/1027752")，如有冒犯，请立即联系删除。

## （4）手写JPEG图像处理引擎

     我们都知道`bitmap`是在`native`层被创建的，在`Bitmap.cpp`文件中，创建的`bitmap`其实是创建了一个`SKBitmap`的对象，交给了skia引擎去处理。

     导入`jpeglib.h`的头文件会需要其他的`.h`头文件，具体如下：

![在这里插入图片描述](file:///Users/mantou/.config/joplin-desktop/resources/6b2e08cf0e3a485fa3dfa1079f9ebff8.webp)

     然后开始撸代码，照着安卓源码中 `libjpeg-turbo`库里的`example.c`文件（系统提供的例子），开始编写`native-lib.cpp`文件：

```
#include <jni.h>
#include <string>
#include <android/bitmap.h>
#include <android/log.h>
#include <malloc.h>
// 因为头文件都是c文件，咱们写的是.cpp 是C++文件，这时候就需要混编，所以加入下面关键字
extern "C"
{
#include "jpeglib.h"
}
#define LOGE(...) __android_log_print(ANDROID_LOG_ERROR,LOG_TAG,__VA_ARGS__)
#define LOG_TAG "louis"
#define true 1
typedef uint8_t BYTE;
// 写入图片函数
void writeImg(BYTE *data, const char *path, int w, int h) {

//  信使： java与C沟通的桥梁，jpeg的结构体，保存的比如宽、高、位深、图片格式等信息
    struct jpeg_compress_struct jpeg_struct;
//  设置错误处理信息 当读完整个文件的时候就会回调my_error_exit，例如内置卡出错、没权限等
    jpeg_error_mgr err;
    jpeg_struct.err = jpeg_std_error(&err);
//  给结构体分配内存
    jpeg_create_compress(&jpeg_struct);
//  打开输出文件
    FILE *file = fopen(path, "wb");
//  设置输出路径
    jpeg_stdio_dest(&jpeg_struct, file);

    jpeg_struct.image_width = w;
    jpeg_struct.image_height = h;
//  初始化  初始化
//  改成FALSE   ---》 开启hufuman算法
    jpeg_struct.arith_code = FALSE;
//  是否采用哈弗曼表数据计算 品质相差2倍多，官方实测， 吹5-10倍的都是扯淡
    jpeg_struct.optimize_coding = TRUE;
//  设置结构体的颜色空间为RGB
    jpeg_struct.in_color_space = JCS_RGB;
//  颜色通道数量
    jpeg_struct.input_components = 3;
//  其他的设置默认
    jpeg_set_defaults(&jpeg_struct);
//  设置质量
    jpeg_set_quality(&jpeg_struct, 60, true);
//  开始压缩，(是否写入全部像素)
    jpeg_start_compress(&jpeg_struct, TRUE);
    JSAMPROW row_pointer[1];
//    一行的rgb
    int row_stride = w * 3;
//  一行一行遍历 如果当前的行数小于图片的高度，就进入循环
    while (jpeg_struct.next_scanline < h) {
//      得到一行的首地址
        row_pointer[0] = &data[jpeg_struct.next_scanline * w * 3];
//		此方法会将jcs.next_scanline加1
        jpeg_write_scanlines(&jpeg_struct, row_pointer, 1);//row_pointer就是一行的首地址，1：写入的行数
    }
    jpeg_finish_compress(&jpeg_struct);
    jpeg_destroy_compress(&jpeg_struct);
    fclose(file);
}

extern "C"
JNIEXPORT void JNICALL
    Java_com_maniu_wechatimagesend_MainActivity_compress(JNIEnv *env, 
                                                    	 jobject instance,
                                                    	 jobject bitmap, 
                                                    	 jstring path_) {

    const char *path = env->GetStringUTFChars(path_, 0);
//  获取Bitmap信息
    AndroidBitmapInfo bitmapInfo;
    AndroidBitmap_getInfo(env, bitmap, &bitmapInfo);
//  存储ARGB所有像素点
    BYTE *pixels;
//  1、读取Bitmap所有像素信息
    AndroidBitmap_lockPixels(env, bitmap, (void **) &pixels);
//  获取bitmap的 宽，高，format
    int h = bitmapInfo.height;
    int w = bitmapInfo.width;
//  存储RGB所有像素点
    BYTE *data,*tmpData;
//  2、解析每个像素，去除A通量，取出RGB通量，
//  假如图片的像素是1920*1080，只有RGB三个颜色通道的话，计算公式为 w*h*3
    data= (BYTE *) malloc(w * h * 3);
//  存储RGB首地址
    tmpData = data;
    BYTE r, g, b;
    int color;
    for (int i = 0; i < h; ++i) {
        for (int j = 0; j < w; ++j) {
            color = *((int *) pixels);
            // 取出R G B 
            r = ((color & 0x00FF0000) >> 16);
            g = ((color & 0x0000FF00) >> 8);
            b = ((color & 0x000000FF));
            // 赋值
            *data = b;
            *(data + 1) = g;
            *(data + 2) = r;
            // 指针后移
            data += 3;
            pixels += 4;
        }
    }
//  3、读取像素点完毕 解锁，
    AndroidBitmap_unlockPixels(env, bitmap);
//  直接用data写数据
    writeImg(tmpData, path, w, h);
    env->ReleaseStringUTFChars(path_, path);
}
```

整个讲解已经在代码里已经做了注释

## （5）总结

**查阅源码发现：**

>      在Android系统在7.0版本之前内部使用的是`libjpeg`非`turbo`版，并且为了性能关闭了`Huffman`编码计算，使用默认的哈夫曼表，而不是算数编码。

>      从 Android 7.0 版本开始，系统内部使用了`libjpeg-turbo`库并且启用`Huffman`编码，标示就是`optimize_code` 已经设置为了 TRUE，也就是默认使用`Huffman`压缩计算生成新的哈夫曼表。`libjpeg-turbo`是一个C语音编写的高效`JPEG`图像处理库，相当于是一个 `libjpeg` 的增强版。

     这也就是`Luban`压缩为什么会给出一个`turbo`分支，其实是为了兼容 **Android 7.0** 版本之前。那么前面也说了，`Luban`压缩可能会造成**OOM**，而且尤其是加载大图的时候，那么怎么去优化呢？让我们一起来看[大图加载优化](https://link.juejin.cn/?target= "https://link.juejin.cn/?target=")。

未完待续~