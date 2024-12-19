---
author: zjmantou
title: Java Type
time: 2024-12-19 周四
tags:
  - Java
  - Type
  - 资料
---
[原文链接](https://www.cnblogs.com/linghu-java/p/8067886.html) 

Type是Java 编程语言中所有类型的公共高级接口（官方解释），也就是Java中所有类型的“爹”；其中，“所有类型”的描述尤为值得关注。它并不是我们平常工作中经常使用的 int、String、List、Map等数据类型，而是从Java语言角度来说，对基本类型、引用类型向上的抽象；

Type体系中类型的包括：原始类型(Class)、参数化类型(ParameterizedType)、数组类型(GenericArrayType)、类型变量(TypeVariable)、基本类型(Class);

原始类型，不仅仅包含我们平常所指的类，还包括枚举、数组、注解等；

参数化类型，就是我们平常所用到的泛型List、Map；

数组类型，并不是我们工作中所使用的数组String[] 、byte[]，而是带有泛型的数组，即T[] ；

基本类型，也就是我们所说的java的基本类型，即int,float,double等

![](https://upload-images.jianshu.io/upload_images/5621908-020f630967a0ebcb.png)

空接口

Type是个空接口，没有定义任何方法，通过多态提高了程序的扩展性，具体实现去看下面的子类；

![](https://upload-images.jianshu.io/upload_images/5621908-907c5d068bc5aa5d.png)

Type体系

查看源码，Type接口下共有4个"儿子"，每一个“儿子”代表着Java中的一种类型；

## 1.ParameterizedType

参数化类型，即泛型；例如：`List<T>`、`Map<K,V>`等带有参数化的对象;

![](https://upload-images.jianshu.io/upload_images/5621908-cf12404e4d33d215.png)

ParameterizedType源码

## 2.TypeVariable

类型变量，即泛型中的变量；例如：T、K、V等变量，可以表示任何类；在这需要强调的是，TypeVariable代表着泛型中的变量，而ParameterizedType则代表整个泛型；

![](https://upload-images.jianshu.io/upload_images/5621908-37b700e5bfd3f20f.png)

TypeVariable源码

## 3.GenericArrayType

泛型数组类型，用来描述ParameterizedType、TypeVariable类型的数组；即`List<T>[]` 、T[]等；

![](https://upload-images.jianshu.io/upload_images/5621908-e51b2bc13f4cc672.png)

GenericArrayType源码

## 4.Class

上三者不同，Class是Type的一个实现类，属于原始类型，是Java反射的基础，对Java类的抽象；

在程序运行期间，每一个类都对应一个Class对象，这个对象包含了类的修饰符、方法，属性、构造等信息，所以我们可以对这个Class对象进行相应的操作，这就是Java的反射；

## 5.WildcardType

泛型表达式（或者通配符表达式），即？ extend Number、？ super Integer这样的表达式；WildcardType虽然是Type的子接口，但却不是Java类型中的一种；

![](https://upload-images.jianshu.io/upload_images/5621908-60b9f33eea9e9530.png)

WildcardType源码

以上，简单介绍了Java-Type的体系；为了解决泛型，JDK1.5版本开始引入Type接口；在此之前，Java中只有原始类型，所有的原始类型都是通过Class进行抽象；有了Type以后，Java的数据类型得到了扩展，从原始类型扩展为参数化类型(ParameterizedType)、数组类型(GenericArrayType)、类型变量(TypeVariable);

下面就用代码的方式，对其中的5大类型：原始类型(Class)、参数化类型(ParameterizedType)、数组类型(GenericArrayType)、类型变量(TypeVariable)、基本类型(Class) 进一步说明；

## 1.ParameterizedType

ParameterizedType表示参数化类型，也就是泛型，例如`List<T>`、`Set<T>`等；

![](https://upload-images.jianshu.io/upload_images/5621908-1c133380acc2d644.png)

ParameterizedType

在ParameterizedType接口中，有3个方法，分别是getActualTypeArguments()、 getRawType()、 getOwnerType();

### 1.1 getActualTypeArguments

获取泛型中的实际类型，可能会存在多个泛型，例如Map<K,V>,所以会返回Type[]数组；

![](https://upload-images.jianshu.io/upload_images/5621908-d0c7ed73eac1a52f.png)

值得注意的是，无论<>中有几层嵌套(List<Map<String,Integer>)，getActualTypeArguments()方法永远都是脱去最外层的<>(也就是List<>)，将口号内的内容（Map<String,Integer>）返回；

我们经常遇到的`List<T>`，通过getActualTypeArguments()方法，得到的返回值是TypeVariableImpl对象，也就是TypeVariable类型(后面介绍);

### 1.2 getRawType

获取声明泛型的类或者接口，也就是泛型中<>前面的那个值；

![](https://upload-images.jianshu.io/upload_images/5621908-ae10a33448af47d4.png)

### 1.3 getOwnerType

通过方法的名称，我们大概了解到，此方法是获取泛型的拥有者，那么拥有者是个什么意思？

Returns a {@code Type} object representing the type that this type    * is a member of.  For example, if this type is {@code O.I},    * return a representation of {@code O}.  （摘自JDK注释）

通过注解，我们得知，“拥有者”表示的含义--内部类的“父类”，通过getOwnerType()方法可以获取到内部类的“拥有者”；例如： Map  就是 Map.Entry<String,String>的拥有者；

![](https://upload-images.jianshu.io/upload_images/5621908-5402d13b2db94560.png)

## 2.GenericArrayType

泛型数组类型，例如`List<String>[]` 、T[]等；

![](https://upload-images.jianshu.io/upload_images/5621908-50ce74f70d8be652.png)

GenericArrayType

在GenericArrayType接口中，仅有1个方法，就是getGenericComponentType()；

### 2.1 getGenericComponentType

返回泛型数组中元素的Type类型，即`List<String>[]` 中的 `List<String>`（ParameterizedTypeImpl）、T[] 中的T（TypeVariableImpl）；

![](https://upload-images.jianshu.io/upload_images/5621908-e7ad6007ce6020f9.png)

值得注意的是，无论是几维数组，getGenericComponentType()方法都只会脱去最右边的[]，返回剩下的值；

## 3.TypeVariable

泛型的类型变量，指的是`List<T>`、`Map<K,V>`中的T，K，V等值，实际的Java类型是TypeVariableImpl（TypeVariable的子类）；此外，还可以对类型变量加上extend限定，这样会有类型变量对应的上限；

![](https://upload-images.jianshu.io/upload_images/5621908-4e4d1d89e59b75ee.png)

TypeVariable

在TypeVariable接口中，有3个方法，分别为getBounds()、getGenericDeclaration()、getName()；

### 3.1 getBounds

获得该类型变量的上限，也就是泛型中extend右边的值；例如 `List<T extends Number>` ，Number就是类型变量T的上限；如果我们只是简单的声明了`List<T>`（无显式定义extends），那么默认为Object；

![](https://upload-images.jianshu.io/upload_images/5621908-be769d3445688382.png) 

无显式定义extends：

![](https://upload-images.jianshu.io/upload_images/5621908-a424cd73be07737e.png)

值得注意的是，类型变量的上限可以为多个，必须使用&符号相连接，例如 List<T extends Number & Serializable>；其中，& 后必须为接口；

### 3.2 getGenericDeclaration

获取声明该类型变量实体，也就是`TypeVariableTest<T>`中的TypeVariableTest；

![](https://upload-images.jianshu.io/upload_images/5621908-edf973323d84f0f2.png)

### 3.3 getName

获取类型变量在源码中定义的名称；

![](https://upload-images.jianshu.io/upload_images/5621908-8ff77ff712bc33eb.png)

说到TypeVariable类，就不得不提及Java-Type体系中另一个比较重要的接口---GenericDeclaration；含义为：声明类型变量的所有实体的公共接口；也就是说该接口定义了哪些地方可以定义类型变量（泛型）；

通过查看源码发现，GenericDeclaration下有三个子类，分别为Class、Method、Constructor；也就是说，我们定义泛型只能在一个类中这3个地方自定义泛型；

![](https://upload-images.jianshu.io/upload_images/5621908-2df89f96f131878f.png)

此时，我们不禁要问，我们不是经常在类中的属性声明泛型吗，怎么Field没有实现 GenericDeclaration接口呢？

其实，我们在Field中并没有声明泛型，而是在使用泛型而已！不信，我们实际上代码来看看！

1.首先在Class上定义泛型：

![](https://upload-images.jianshu.io/upload_images/5621908-6e53fb35c8715da3.png)

Class定义泛型

2.我们没有在Class上定义泛型，直接在构造方法上定义泛型

![](https://upload-images.jianshu.io/upload_images/5621908-1de41c143f0d4823.png)

泛型构造

3.同样没有在Class定义泛型，直接在普通方法上定义泛型

![](https://upload-images.jianshu.io/upload_images/5621908-e0dcbbd0e8eab288.png)

泛型方法

3.我们直接在属性上定义

![](https://upload-images.jianshu.io/upload_images/5621908-9e57982e43e00fe8.png)

属性上定义泛型

我们看到，如果不在Class上定义，属性上并不能直接使用！所以，这也是我之前说的属性上并不是定义泛型，而是使用泛型，所以Field并没有实现GenericDeclaration接口！

## 4.Class

Type接口的实现类，是我们工作中常用到的一个对象；在Java中，每个.class文件在程序运行期间，都对应着一个Class对象，这个对象保存有这个类的全部信息；因此，Class对象也称之为Java反射的基础；

![](https://upload-images.jianshu.io/upload_images/5621908-1c78a23eb02d33d3.png)

Class

通过上面的例子，可以看出，当我们没有声明泛型的时候，我们普通的对象就是一个Class类型，是Type中的一种；

## 5.WildcardType

？---通配符表达式，表示通配符泛型，但是WildcardType并不属于Java-Type中的一钟；例如：`List<? extends Number> 和 List<? super Integer>`；

![](https://upload-images.jianshu.io/upload_images/5621908-2de326cc6e08c440.png)

WildcardType

在WildcardType接口中，有2个方法，分别为getUpperBounds()、getLowerBounds();

### 5.1 getUpperBounds

获取泛型变量的上边界（extends）

![](https://upload-images.jianshu.io/upload_images/5621908-391dfb086762db77.png)

### 5.2 getLowerBounds

获取泛型变量的下边界（super）

![](https://upload-images.jianshu.io/upload_images/5621908-fd3cb71089452e2f.png)

以上，就是对Java-Type体系中相关对象的介绍；

# 总结 

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202412191500782.png)
