
#### Servcie保活

- 在onStartCommend()将返回值设置为START_STICKY
- 在onDestroy()中重启
- 其他进程唤起
- 启动前台service
- 提高service优先级

#### 反编译

- apktools：反编译成原始目录，可以看到清单文件；
- dex2jar：将dex文件转化成一个classes.jar文件；
- jd-gui：将classes.jar转换为.java的源代码；

#### Android中哪些用到了Binder

在Android系统中，很多组件和模块都使用了Binder作为进程间通信（IPC）方式。以下是一些使用了Binder的组件和模块：

1. Activity、Service、Broadcast、ContentProvider是Android系统中的四个主要组件，它们在多进程通信时底层都依赖于Binder IPC机制。例如，当进程A中的Activity要与进程B中的Service通信时，就需要依赖Binder IPC。
2. 调用系统服务，如获取输入法服务、闹钟服务、摄像头、电话等系统服务，都会用到进程间通信Binder。
3. 对于一些吃内存的模块，如地图模块、大图浏览、webview等，由于Android对每个进程的内存空间有限制，因此也使用了Binder机制进行进程间通信。

总之，Binder是Android系统中广泛使用的进程间通信机制，几乎所有的Android应用程序都会使用到它。

#### 图片压缩方案

![[图片压缩方案#基础知识#小结]]

- 质量压缩：在不改变图片尺寸的情况下，改变图片的存储体积。
	- compress
- 采样压缩：是降低图像尺寸，达到相同目的。
	- 邻近采样：inSampleSize
	- 双线性采样：createBitmap，可以使用小数，处理文字上效果更好
	- 双三次采样
	- Lanczos：计算量最大。


![[图片压缩方案#Android中图片压缩的方法介绍#小结]]