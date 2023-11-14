---
author: zjmantou
title: 06Class类文件的结构
time: 2023-11-14 周二
tags:
  - Java
  - Java虚拟机
---
# 基础概念

**整个class文件本质上是一张表**

## 简单名称

没有类型和参数修饰的方法或者字段名称，这个类中的inc()方法和m字段的简单名称分别是“inc”和“m”。

## 描述符号

用来描述字段的数据类型、方法的参数列表（包括数量、类型以及顺序）和返回值。

规则：
- 基本数据类型以及void类型用一个开头字母大写表示
- 对象类型：L+对象全限定名+;表示
- 数组：前面加个`[`；
- 方法：前面加个`()`
- 方法中有参数，则在`()`里面按照参数类型的顺序加。

![描述符标识自负含义.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142247096.jpeg)


## 全限定名

把类全名中的"."换成“/”，比如“org/fenixsoft/clazz/TestClass”，再最后加一个分号“;”。


# 类结构

Class文件是一组以8位字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑地排列在Class文件之中，中间没有添加任何分隔符，当遇到需要占用8位字节以上空间的数据项时，则会按照高位在前￼的方式分割成若干个8位字节进行存储。 

Class文件格式采用一种类似C语言结构题的伪结构来存储数据：
- 无符号数，属于基本数据类型
	- 以u1、u2、u4、u8分别代表1个字节、2个字节、4个字节、8个字节的无符号数
	- 可以用来描述数字、索引引用、数量值或按照UTF-8编码构成的字符串值
- 表：由多个无符号数或者其他表作为数据项构成的复合数据类型
	- 习惯性的以`_info`结尾
	- 用于描述有层次关系的复合结构的数据
	- 整个class文件本质上是一张表

![Class文件格式.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311141711707.png)

## 魔数

每个Class文件的头4个字节称为魔数（Magic Number），它的唯一作用是确定这个文件是否为一个能被虚拟机接受的Class文件。 

Java的Class文件魔数获得：0xCAFEBABE。  

紧接着魔数的4个字节存储的是Class文件的版本号：第5和第6个字节是次版本号（Minor Version），第7和第8个字节是主版本号（Major Version）。

## 常量池（contant_pool）

可以理解为Class文件中的资源仓库它是Class文件结构中与其他项目关联最多的数据类型，也是占用Class文件空间最大的数据项目之一，同时它还是在Class文件中第一个出现的表类型数据项。 

入口u2数据类型：代表常量池容量计数值（constant_pool_count）从1开始。

比如容量为22，则表示有21项常量，索引值范围为1～21. 

0用来表达不引用任何一个常量池项目。 

主要存放两大类：
- 字面量（Literal）
	- 文本字符串、final声明的常量值
- 符号引用（Symbolic References）
	- 类和接口的全限定名（Fully Qualified Name）
	- 字段的名称和描述符（Descriptor）
	- 方法的名称和描述符

Java代码在进行Javac编译的时候，并不像C和C++那样有“连接”这一步骤，而是在虚拟机加载Class文件的时候进行动态连接。

![6-3 常量池的项目类型.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142223727.png)


### 常量池结构总表

![9EA72DD2-1881-4ED6-B36B-917C5F98FA6E.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142229405.png)


## 访问标志（access_flag）

用于识别一些类或者接口层次的访问信息，包括：这个Class是类还是接口；是否定义为public类型；是否定义为abstract类型；如果是类的话，是否被声明为final等。

![B87C5B98-04CE-4E8B-8AEC-E345C9346987_4_5005_c.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142232081.jpeg)


比如一个类它的ACC_PUBLIC和ACC_SUPER为真，access_flags的值为：0x0001|0x0020=0x0021。 

## 类索引、父类索引与接口索引集合

类索引（this_class）和父类索引（super_class）都是一个u2类型的数据，而接口索引集合（interfaces）是一组u2类型的数据的集合，Class文件中由这三项数据来确定这个类的继承关系。类索引用于确定这个类的全限定名，父类索引用于确定这个类的父类的全限定名。

接口索引集合，入口的第一项——u2类型的数据为接口计数器（interfaces_count）

## 字段表（field_info）集合

![字段表结构.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142240638.jpeg)

- name_index：字段的简单名称
- descriptor_index：字段和方法的[[06Class类文件的结构#基础概念#描述符号|描述符]]

### 字段表访问标志

![BDA5B66F-AF7A-4739-9A60-17C654DCCC87_4_5005_c.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142242824.jpeg)


## 方法表集合

![方法表结构实例.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142258380.jpeg)

### 方法表结构

![方法表结构.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142255714.jpeg)

### 方法访问标志（access_flags）

![方法访问标志.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142256298.png)

## 属性表集合（attribute_info）

![虚拟机规范预定义的属性.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142300582.png)


### 1.Code属性

Java程序方法体中的代码经过Javac编译器处理后，最终变为字节码指令存储在Code属性内。 

![Code属性表的结构.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142301784.jpeg)


- attribute_name_index：代表属性名称，指向CONSTANT_Utf8_info型常量的索引，固定值为“Code”；
- 属性值的长度：属性名称和属性值的长度共6个字节，所以属性值的长度为属性表长度减去6个字节；
- max_stack：操作数栈（perand Stacks）深度的最大值。虚拟机的时候根据这个值来分配栈帧中的操作栈深度；
- max_locals：局部变量表所需的存储空间。单位是Slot；
	- Slot可以重用
- code_length、Code：存储遍以后生成的字节码指令；最大不超过65535条（64KB）字节码指令[[附录B“虚拟机字节码指令表”]]

>字节码指令中，普通无参方法的Locals和Args_size都为1，因为有个this关键字，只有static方法才会0；

#### 异常表

在字节码指令之后的是这个方法的显式异常处理表（下文简称异常表）集合

![异常表.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142313271.jpeg)

- start_pc、end_pc：字节码在第start_pc行到end_pc（不包含end_pc）行出现异常，类似try代码块；
- catch_type：异常类型，指向CONSTANT_Class_info型常量索引；
- handler_pc：异常情况转到handler_pc处处理。

### 2.Exception属性

与Code属性平级的一项属性，跟异常表不一样；

作用是列举出方法中可能抛出的受查异常（Checked Exceptions），就是方法在throws关键字后面的异常；

![属性表结构.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142319356.jpeg)

- number_of_exceptions：可能抛出多少钟异常；
- exception_index_table：指向CONSTANT_Class_info型常量的索引

### 3.LineNumberTable属性（可选）

LineNumberTable属性用于描述Java源码行号与字节码行号（字节码的偏移量）之间的对应关系。

可以在Javac中分别使用-g:none或-g:lines选项来取消或要求生成这项信息。 

如果取消，则运行时抛出异常将不会显示行号。 

可通过Javac中的-g:none或-g:lines来取消.  

![LineNumberTable属性结构.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142323094.jpeg)

line_number_table是一个数量为line_number_table_length、类型为line_number_info的集合，line_number_info表包括了start_pc和line_number两个u2类型的数据项，前者是字节码行号，后者是Java源码行号。

### 4.LocalVariableTable属性（可选）

用于描述栈帧中局部变量表中的变量与Java源码中定义的变量之间的关系。

可通过Javac中的-g:none或-g:vars来取消. 

![LocalVariableTable属性结构.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142325406.jpeg)

#### local_variable_info

代表了一个栈帧与源码中的局部变量的关联  

![ocal_variable_info.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142326906.jpeg)

- start_pc、length：这个局部变量的生命周期开始的字节码偏移量及其作用范围覆盖的长度；作用域范围；
- name_index：局部变量名称；
- descriptor_index：局部变量描述符；
- index：这个局部变量在栈帧局部变量表中Slot的位置；

### 5.SourceFile属性（可选）

用于记录生成这个Class文件的源码文件名称。

可通过Javac中的-g:none或-g:source来关闭。 

![sourcefile属性结构.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142331307.jpeg)

sourcefile_index数据项是指向常量池中CONSTANT_Utf8_info型常量的索引，常量值是源码文件的文件名。 

### 6.ConstantValue属性 

通知虚拟机自动为静态变量赋值。 

- 非static类型变量：在实例构造器`<init>`方法中
- 静态变量：
	- 类构造器`<clinit>`
	- 使用ConstantValue属性。 

![ConstantValue属性结构.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142335223.jpeg)


### 7.InnerClasses属性

记录内部类与宿主类之间的关联 

![InnerClasses属性结构.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142337912.jpeg)

- number_of_classes：需要记录多少个内部类信息；

#### inner_classes_info表

![inner_classes_info表.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142338245.jpeg)

- inner_class_info_index：内部类索引
- outer_class_info_index：外部类索引
- inner_name_index：内部类名称，匿名内部类值为0；
- inner_class_access_flags：内部类的访问标志；
#### inner_class_access_flags标志

![ inner_class_access_flags标志结构.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142340263.jpeg)


### 8.Deprecated及Synthetic属性

属于标志类型的布尔属性。  

- Deprecated：表示已经舍弃，不再推荐
- Synthetic：此字段或方法不是有Java源码直接产生的，而是由编译器自行添加（1.5之后可以用ACC_SYNTHETIC标志位） 

![Deprecated及Synthetic属性结构.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142343008.jpeg)


### 9.StackMapTable属性

1.6后增加，在验证阶段被新类型检查验证器（Type Checker）使用。  

包含0到多个栈映射帧（Stack Map Frames），每个栈映射帧都显式或隐式地代表了一个字节码偏移量，用于表示该执行到该字节码时局部变量表和操作数栈的验证类型。

![StackMapTable属性的结构.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142346034.jpeg)

>《Java虚拟机规范（Java SE 7版）》明确规定：在版本号大于或等于50.0的Class文件中，如果方法的Code属性中没有附带StackMapTable属性，那就意味着它带有一个隐式的StackMap属性。这个StackMap属性的作用等同于number_of_entries值为0的StackMapTable属性。一个方法的Code属性最多只能有一个StackMapTable属性，否则将抛出ClassFormatError异常。


### 10.Signature属性

JDK1.5之后增加，泛型签名记录，为了解决泛型擦除缺陷。 

因为Java中泛型擦除；

泛型擦除：
- 好处：实现简单、能够节省运行期内存空间。 
- 坏处：伪泛型，无法在运行期将泛型类型与用户定义的普通类型同等对待，如运行期做反射无法获取泛型信息。

![Signature属性结构.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142351548.jpeg)

### 11.BootstrapMethods属性

JDK1.7之后发布。 

![7DCB5F47-CD48-40CB-A4B8-A5987F5CC679_4_5005_c.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142352082.jpeg)


![F68B0386-57BD-4D2A-B3CF-D6E20471581C_4_5005_c.jpeg](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311142353189.jpeg)
