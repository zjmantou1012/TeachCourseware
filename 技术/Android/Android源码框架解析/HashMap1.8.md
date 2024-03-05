---
author: zjmantou
title: HashMap1.8
time: 2024-03-05 周二
tags:
  - Java
  - 技术
  - 源码
---
# hash
```java
static final int hash(Object key) {
       int h;
       return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
   }
```

## 为什么要把hash值右移16位？

[HashMap中的hash方法为什么要右移16位并异或？](https://blog.csdn.net/qq_34115899/article/details/126367040)

```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;          
        // .......源码自行查看，只展示关键部分
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        // .......源码自行查看，只展示关键部分
    }
```

put时数组是通过hash值与数组长度取模得到，`i = (n - 1) & hash`，n是2的幂次方，在2^16（65536）范围内，2的幂次方是为了更好的分散，因为n-1的二进制是`1111...11`，而&造成最高只取低16位，散列度不高，为了提高散列度，hash做了位移运算，将高位移到低位，然后异或后散列度更高，相当于将高位和低位组合。

# Put操作

1. 如果是null就先扩容初始化；
2. 获取长度判断是否要扩容（table是个懒加载，等第一个put之后才开始加载）；
3. 如果计算出来的hash桶没有值，则直接插入；
4. 如果有，则是存在hash冲突：
	1. 如果插入的hash与key都与当前节点相等，则该节点为首节点；
	2. 判断是否是树节点，如果是则putTreeVal在红黑树中进行添加；
	3. 不是树节点，则遍历链表：
		1. 如果是尾节点（next==null）则直接添加到尾部，判断是否要树化；
		2. 如果有重复的key，则替换；
	4. 如果有重复的值，则替换后返回旧值；
	5. 如果没有重复，则+1，判断是否要扩容；

# 扩容

1. 如果没有初始化，则初始化（16）
2. 扩容为原来的2倍（前提不能大于最大值1<<30）
3. 移动旧表到新表中：
	1. 如果只有一个首节点，将他放入新表中
	2. 如果是红黑树，调用split拆分
	3. 链表结构，遍历值转移到新的地方，并把旧地址置空；

**这里原来的数组的地址还是会放在原来的下标**，因为新的值是`e.hash & (newCap - 1)`



![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403052035591.png)



# 参考链接 

[深入理解HashMap原理(一)——HashMap源码解析(JDK 1.8)](https://ost.51cto.com/posts/952)
