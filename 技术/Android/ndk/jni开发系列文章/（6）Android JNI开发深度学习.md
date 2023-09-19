---
author: zjmantou
title: （6）Android JNI开发深度学习
time: 2023-09-20 周三
tags:
  - Android
  - NDK
  - JNI
---
**JNI开发系列目录**

1. [JNI开发必学C++基础](https://blog.csdn.net/luo_boke/article/details/126908373 "https://blog.csdn.net/luo_boke/article/details/126908373")
2. [JNI开发必学C++使用实践](https://blog.csdn.net/luo_boke/article/details/126920916 "https://blog.csdn.net/luo_boke/article/details/126920916")
3. [Android Studio 4.0.+NDK项目开发详细教学](https://blog.csdn.net/luo_boke/article/details/107306531 "https://blog.csdn.net/luo_boke/article/details/107306531")
4. [Android NDK与JNI的区别有何不同？](https://blog.csdn.net/luo_boke/article/details/107234358 "https://blog.csdn.net/luo_boke/article/details/107234358")
5. [Android Studio 4.0.+NDK .so库生成打包](https://blog.csdn.net/luo_boke/article/details/109362013 "https://blog.csdn.net/luo_boke/article/details/109362013")
6. [Android JNI的深度进阶学习](https://blog.csdn.net/luo_boke/article/details/109455910 "https://blog.csdn.net/luo_boke/article/details/109455910")
7. [Android Studio 4.0.+NDK开发 This files is not part of the project](https://blog.csdn.net/luo_boke/article/details/109318721 "https://blog.csdn.net/luo_boke/article/details/109318721")

---

**`以Android studio 4.0.2来分析讲解，gradle=6.1.1，如图文和网上其他资料不一致，可能是别的资料版本较低而已`**

---

## 前言

JNI的全称是Java [Native](https://so.csdn.net/so/search?q=Native&spm=1001.2101.3001.7020 "https://so.csdn.net/so/search?q=Native&spm=1001.2101.3001.7020") Interface，即本地Java接口。采用JNI特性可以增强 Java 与本地代码交互的能力，使Java和其他类型的语言如C++/C能够互相调用。  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200005409.png)


---

## JNI原理

Java语言的执行环境是Java虚拟机(JVM)，JVM其实是主机环境中的一个进程，每个JVM虚拟机都在本地环境中有一个JavaVM结构体，该结构体在创建JVM虚拟机时被返回。JNI全局仅仅有一个，JavaVM是Java虚拟机在JNI层的代表，一个JVM对应一个JavaVM结构。

一个JVM中可能创建多个Java线程，每个线程对应一个JNIEnv结构，它们保存在线程本地存储TLS中。JNIEnv是一个线程相关的函数表结构体，该结构体代表了Java在本线程的执行环境。

不同的线程的JNIEnv是不同，也不能相互共享使用。在本地代码中通过JNIEnv的函数表来操作Java数据或者调用Java方法。

---

## JNI函数创建

在AS中如申明一个native方法，AS可以自动帮.cpp文件中创建一个native函数  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200005708.png)


```
#include <jni.h>

extern "C"
JNIEXPORT jint JNICALL
Java_com_xuanyuan_ndktest_MainActivity_add(JNIEnv *env, jobject thiz, jint a, jint b) {
    // TODO: implement add()
}
```

对于如上的JNI函数我们一项项分析

### **extern “C”**

指定以"C"的方式来实现native函数，当然你也可以选择用`extern "C++"`。两种方式大致一样，主要是对env的操作方式略有区别

```
extern "C++" JNIEXPORT jstring JNICALL Java_com_szysky_note_androiddevseek_114_JNITest_get(JNIEnv *env, jobject thiz){
    printf("执行在c++文件中 get方法\n");
    return env->NewStringUTF("Hello from JNI .");
}

extern "C" JNIEXPORT jstring JNICALL Java_com_szysky_note_androiddevseek_114_JNITest_get(JNIEnv *env, jobject thiz){
    printf("执行在c文件中 get方法\n");
    return (*env)->NewStringUTF("Hello from JNI .");
}


//区别：
C++: env->ReleaseStringUTFChars(string, str);
C:  (*env)->ReleaseStringUTFChars(env, string, str); 

```

### **JNIEXPORT**  
宏定义，用于指定该函数是JNI函数。表示此函数可以被外部调用，在Android开发中`不可省略`

### **JNICALL**  
宏定义，用于指定该函数是JNI函数。，无实际意义，但是`不可省略`

### **JNIEnv env**  
JNIEnv 代表了JNI的环境，只要在本地代码中拿到了JNIEnv和jobject，JNI层实现的方法都是通过JNIEnv 指针调用JNI层的方法访问Java虚拟机，进而操作Java对象，这样就能调用Java代码了。

### **jobject thiz**  
在AS中自动为我们生成的JNI方法声明都会带一个这样的参数，这个instance就代表Java中native方法声明所在的类，比如上面add(int a,int b)方法声明在MainActivity中，这里的instance就表示MainActivity实例。

---

## Java与C/C++互相调用

java和C/C++是可以互相调用，下面我们分别分析两种情况

```
// Java代码
public class MainActivity extends AppCompatActivity {
    static {
        System.loadLibrary("native-lib");
    }
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // Example of a call to a native method
        TextView tv = findViewById(R.id.sample_text);
        tv.setText(new MyNdkTest().getData());
    }
}

***************************************************************
// C代码
extern "C"
JNIEXPORT jstring JNICALL
Java_com_xuanyuan_ndktest_MyNdkTest_getData(JNIEnv *env, jobject thiz) {
    std::string hello = "我还会回来的！";
    return env->NewStringUTF(hello.c_str());
}


```

### **1. Java调用C/C++函数**  
**调用流程：** Java层调用某个函数时，会从对应的JNI层中寻找该函数。根据java函数的包名、方法名、参数列表等多方面来确定函数是否存在。如果没有就会报错，如果存在就会就会建立一个关联关系，以后再调用时会直接使用这个函数，这部分的操作由虚拟机完成。  
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200010616.png)


例如在MainActivity 中调用MyNdkTest类的native getData()方法，程序会自动在JNI层查找Java_com_xuanyuan_ndktest_MyNdkTest_getData函数接口，如未找到则报错。如找到，则会调用native库中的对应函数。

---

### **2. C/C++函数调用Java**

在JNI函数中总会有一个参数jobject thiz，它代表着调用该JNI函数的类的实例，这里是MainActivity的实例。通过JNIEnv env和jobject thiz就调用MainActivity中的函数和字段。

---

## JNI开发细则

**1. JNI函数命名规则：**

1. 本地代码函数如果后缀为.h，则方法要由`extern "C" { }`包裹，.cpp文件不用
2. `JNIEXPORT jstring JNICALL`中的`JNIEXPORT` 和 `JNICALL`不能省，且jstring是JNI的一种数据类型，相当于Java中的String
3. 如果在Java中声明的方法是"静态的"，则native方法也是static。
4. JNI函数的命名规则为：Java_包名_类名_方法名。包名里的`.`要改成`_`，`_`要改成`_1`
5. 如果你的JNI的native方法不是通过静态注册方式来实现的，则不需要符合上面的这些规范，可以格局自己习惯随意命名

---

**2. 数据类型**

因为Java层和C/C++的数据类型或者对象不能直接相互的引用或者使用，JNI层定义了自己的数据类型，用于衔接Java层和JNI层。其一 一对应关系如下。

|Java类型|JNI类型|Java类型|JNI类型|
|---|---|---|---|
|boolean|jboolean|boolean[]|jbooleanArray|
|byte|jbyte|byte[]|jbyteArray|
|char|jchar|char[]|jcharArray|
|short|jshort|short[]|jshortArray|
|int|jint|int[]|jintArray|
|long|jlong|long[]|jlongArray|
|float|jfloat|float[]|jfloatArray|
|double|jdouble|double[]|jdoubleArray|
|Object|jobject|Object[]|jobjectArray|
|Class|jclass|||
|String|jstring|||

---

**3. 常用方法**  
`1. NewObject(JNIEnv *env, jclass clazz,jmethodID methodID, ...)`  
创建一个对象

`2. string NewString(JNIEnv *env, const jchar *unicodeChars,jsize len)`  
创建一个新的String对象

`3. ArrayType New<PrimitiveType>Array(JNIEnv *env, jsize length)`  
各种类型的数组

`4.jobjectArray NewObjectArray(JNIEnv *env, jsize length,jclass elementClass, jobject initialElement)`  
创建类型为elementClass的对象数组，其数组值初始化为initialElement

`5. jobject GetObjectArrayElement(JNIEnv *env,jobjectArray array, jsize index)`  
从指定数组中获得其中某个位置的元素

`6. jsize GetArrayLength(JNIEnv *env, jarray array)`  
获取array数组的长度

`7. 获取Class对象`  
为了能够在C/C++中调用Java中的类，jni.h的头文件专门定义了jclass类型表示Java中Class类。JNIEnv中有3个函数可以获取jclass。

- **jclass FindClass(const char* clsName)** ：  
    通过类的名称(类的全名，这时候包名不是用’".“点号而是用”/"来区分的)来获取jclass。比如: jclass jcl_string=env->FindClass(“java/lang/String”);
    
- **class GetObjectClass(jobject obj)**：  
    通过对象实例来获取jclass，相当于Java中的getClass()函数
    
- **jclass getSuperClass(jclass obj)**：  
    通过jclass可以获取其父类的jclass对象
    

`8. 获取属性及方法`  
在Native本地代码中访问Java层的代码，一个常用的常见的场景就是获取Java类的属性和方法。所以为了在C/C++获取Java层的属性和方法，JNI在jni.h头文件中定义了jfieldID和jmethodID这两种类型来分别代表Java端的属性和方法。

在访问或者设置Java某个属性的时候，首先就要现在本地代码中取得代表该Java类的属性的jfieldID，然后才能在本地代码中进行Java属性的操作，同样，在需要调用Java类的某个方法时，也是需要取得代表该方法的jmethodID才能进行Java方法操作。

```
jfieldID GetFieldID(JNIEnv *env, jclass clazz, const char *name, const char *sig);
jmethodID GetMethodID(JNIEnv *env, jclass clazz, const char *name, const char *sig);
jfieldID GetStaticFieldID(JNIEnv *env, jclass clazz, const char *name, const char *sig);
jmethodID GetStaticMethodID(JNIEnv *env, jclass clazz,const char *name, const char *sig);

JNIEnv代表一个JNI环境接口，jclass上面也说了代表Java层中的"类"，name则代表方法名或者属性名。那最后一个char *sig代表了JNI中的一个特殊字段——签名，

```

---

**4. JNI引用类型**

从java 虚拟机中创建的对象传到C/C++代码中会产生引用，根据Java的垃圾回收机制，只要有引用存在就不会触发该引用所指向Java对象的垃圾回收，所以在JNI调用参数需要做一些额外处理。

在JNI规范中定义了三种引用：局部引用（Local Reference）、全局引用（Global Reference）、弱全局引用（Weak Global Reference）。

`1.局部引用(Local Reference)`

局部引用，也称本地引用，通常是在函数中创建并使用。会阻止GC回收所有引用对象。在函数中产生的局部引用，都会在函数返回的时候自动释放(freed)，也可以使用DeleteLocalRef函数手动释放该应用。

`2.全局引用(Global Reference)`

全局引用可以跨方法、跨线程使用，直到被开发者显式释放。一个全局引用在被释放前保证引用对象不被GC回收。和局部应用不同的是，能创建全局引用的函数只有NewGlobalRef，而释放它需要使用ReleaseGlobalRef函数

`3. 弱全局引用(Weak Global Reference)`

与全局引用类似，创建跟删除都需要由编程人员来进行，这种引用与全局引用一样可以在多个地方有效。通过使用NewWeakGlobalRef、ReleaseWeakGlobalRef来产生和解除引用。

**`注意：和全局引用不一样的是，弱引用将不会阻止垃圾回收器回收这个引用所指向的对象，所以在使用时需要多加小心，它所引用的对象可能是不存在的或者已经被回收。`**

---

**5. 注册native函数**

当Java代码中执行Native的代码的时候，首先是通过一定的方法来找到这些native方法。而注册native函数的具体方法不同，会导致系统在运行时采用不同的方式来寻找这些native方法。JNI有如下两种注册native方法的途径：静态注册与动态注册

**1. 静态注册**

先由Java得到本地方法的声明，然后再通过JNI实现该声明方法。

静态注册就是根据函数名来遍历Java和JNI函数之间的关联，而且要求JNI层函数的名字必须遵循特定的格式。具体的实现很简单，首先在Java代码中声明native函数，然后通过javah来生成native函数的具体形式，接下来在JNI代码中实现这些函数即可。

**2. 动态注册**

先通过JNI重载JNI_OnLoad()实现本地方法，然后直接在Java中调用本地方法。

通过RegisterNatives方法把C/C++中的方法映射到Java中的native方法，而无需遵循特定的方法命名格式，这样书写起来会省事很多。

当我们使用System.loadLibarary()方法加载so库的时候，Java虚拟机就会找到这个JNI_OnLoad函数兵调用该函数，这个函数的作用是告诉Dalvik虚拟机此C库使用的是哪一个JNI版本，如果你的库里面没有写明JNI_OnLoad()函数，VM会默认该库使用最老的JNI 1.1版本。

由于最新版本的JNI做了很多扩充，也优化了一些内容，如果需要使用JNI新版本的功能，就必须在JNI_OnLoad()函数声明JNI的版本。同时也可以在该函数中做一些初始化的动作，其实这个函数有点类似于Android中的Activity中的onCreate()方法。

与JNI_OnLoad()函数相对应的有JNI_OnUnload()函数，当虚拟机释放该C库的时候，则会调用JNI_OnUnload()函数来进行善后清除工作。

---

## JNI debug开启

1. Debug type修改为Dual(Java+Native)  
   ![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200010230.png)

    
2. Build Variants->Jni Debuggable 改为true  
    ![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309200011502.png)

    

---

**相关链接**：  
3. [JNI开发必学C++基础](https://blog.csdn.net/luo_boke/article/details/126908373 "https://blog.csdn.net/luo_boke/article/details/126908373")  
4. [JNI开发必学C++使用实践](https://blog.csdn.net/luo_boke/article/details/126920916 "https://blog.csdn.net/luo_boke/article/details/126920916")  
5. [Android Studio 4.0.+NDK项目开发详细教学](https://blog.csdn.net/luo_boke/article/details/107306531 "https://blog.csdn.net/luo_boke/article/details/107306531")  
6. [Android NDK与JNI的区别有何不同？](https://blog.csdn.net/luo_boke/article/details/107234358 "https://blog.csdn.net/luo_boke/article/details/107234358")  
7. [Android Studio 4.0.+NDK .so库生成打包](https://blog.csdn.net/luo_boke/article/details/109362013 "https://blog.csdn.net/luo_boke/article/details/109362013")  
8. [Android JNI的深度进阶学习](https://blog.csdn.net/luo_boke/article/details/109455910 "https://blog.csdn.net/luo_boke/article/details/109455910")  
9. [Android Studio 4.0.+NDK开发 This files is not part of the project](https://blog.csdn.net/luo_boke/article/details/109318721 "https://blog.csdn.net/luo_boke/article/details/109318721")

**博客书写不易，您的点赞收藏是我前进的动力，千万别忘记点赞、 收藏 ^ _ ^ !**