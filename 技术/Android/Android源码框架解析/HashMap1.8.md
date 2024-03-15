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

***注：1.8不需要rehash***

因为扩容之后就是原来容量的2倍，比如16扩容到32，则长度就是`0000 1000`变成了 `0001 0000`，rehash的值与上长度n-1，其实就是高位进1，因此，扩容后rehash无非就是两种情况：
1. ***值为原来的值，则放在原来的低位位置***；
2. ***高位取模为1，则需要将该节点放在高位，即原来位置+旧的容量***

1.8源码关键的方法：`if ((e.hash & oldCap) == 0)` 

**正是通过原来hash的值与原数组的最高位取模判断是否为0来得到该节点是需要放在低位还是高位数组。**

## 源码

#### 解读资料

[JDK1.8中HashMap扩容机制](https://blog.csdn.net/yueaini10000/article/details/109030129)

```java
/**
 * JDK1.8---哈希表扩容
 * @return
 */
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    /**
     * 获取原哈希表容量  如果哈希表为空则容量为0 ，否则为原哈希表长度
     */
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    /**
     * 获取原哈希表扩容门槛
     */
    int oldThr = threshold;
    /**
     * 初始化新容量和新扩容门槛为0
     */
    int newCap, newThr = 0;
    /**
     //如果原容量大于 0
     ---这个if语句中计算进行扩容后的容量及新的负载门槛
     */
    if (oldCap > 0) {
            //判断原容量是否大于等于HashMap允许的容量最大值 2的30次幂
            if (oldCap >= MAXIMUM_CAPACITY) {
                //如果原容量已经大于等于了允许的最大容量，
                // 则把当前HashMap的扩容门槛设置为Integer允许的最大值
                    threshold = Integer.MAX_VALUE;
                    return oldTab;//不再扩容直接返回
                }
            /**
             * newCap = oldCap << 1 ;  类似 newCap = oldCap * 2 移位操作更加高效
             * 表示把原容量的二进制位向左移动一位，
             * 扩大为原来的2倍，同样还是2的n次幂
             * 如果新的数组容量小于HashMap允许的容量最大值 2的30次幂
             * 并且 原数组容量小于默认的初始化数组容量 2的4次幂 =16
              */
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                             oldCap >= DEFAULT_INITIAL_CAPACITY)
            /**
             * //新的扩容门槛为原来的扩容门槛的2倍，同样二进制左移操作
             //类似 newThr = oldThr * 2 移位操作更加高效
             */
                newThr = oldThr << 1; // double threshold
        }
    /**
     * 如果 原数组容量小于等于零
     * 并且 原负载门槛大于0 则
     * 新数组容量为原负载门槛大小
      */
    else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
    /**
     * 这个elese语句  初始化默认容量和默认负载门槛
     * 如果原数组容量小于等于0
     * 并且原负载门槛也小于等于0
     * 则
     * 新 数组容量为  默认HashMap设置的默认初始化容量  1《4 = 2的4次幂 = 16
     * 新 负载门槛为  默认负载因子（0.75f） * 16;
     */
    else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
    /**
     * 如果新负载门槛为 0  则开始使用新的 数组容量进行计算
      */
    if (newThr == 0) {
           // 新的数组容量 * 负载因子
            float ft = (float)newCap * loadFactor;
        /**
         * 如果新数组容量 小于 HashMap允许的最大容量(2的30次幂)
         * 并且  新计算的负载门槛 小于 HashMap允许的最大容量(2的30次幂)
         * 则新的 负载门槛为 计算后的值 的最大整型 -直接截取
         * 否则  新的负载门槛为Integer.MAX_VALUE
         */
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                              (int)ft : Integer.MAX_VALUE);
        }
    //设置全局负载门槛为计算后的新的负载门槛
    threshold = newThr;
    /**
     * 根据新的数组容量创建新的哈希桶 赋值给newTab
     */
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    /**
     * 把新创建的哈希桶赋值给全局table;
     */
    table = newTab;
    /**
     * 现在开始真正的扩容
     */
    if (oldTab != null) {//如果老的哈希表不为空则执行以下语句
           //for 循环，循环老的容量次
            for (int j = 0; j < oldCap; ++j) {
                    Node<K,V> e;
                /**
                 *   //从哈希数组的第一个下标（0）开始开始递增
                 *         注释：
                 *          e = oldTab[0]  ;
                 *          e = oldTab[1]  ;  循环访问每次哈希数组下标的内容
                 *   e = oldTab[j];
                 *   如果 e != null 则开始访问数组中的内容
                 */
                if ((e = oldTab[j]) != null) {
                            把原数组中下标为j的位置置空
                            oldTab[j] = null;
                            //e.next == null 则代表线性链表只有一个元素e
                            if (e.next == null)
                                /**
                                 * //根据e 的哈希值和 （新数组容量 -1）相
                                 * 与得到 e该存放到新数组中的下标
                                 * 然后把e放入对应新数组的下标中。
                                 */
                                newTab[e.hash & (newCap - 1)] = e;
                            else if (e instanceof TreeNode)
                                /**
                                 * //如果当前e的类型已经改变为红黑树
                                 * 则对红黑树进行分割  ？
                                 */
                                ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                            else { // preserve order
                                /**
                                 * 进入这个else循环代表当前数组下标中存放的元素还是线性链表
                                 */
                                    Node<K,V> loHead = null, loTail = null;//定义两个指针，分别指向低位头部和低位尾部
                                    Node<K,V> hiHead = null, hiTail = null;//定义两个指针，分别指向高位头部和高位尾部
                                    Node<K,V> next;
                                /**
                                 * do-while循环中针对数组下标为j的 
                                 * 线性链表进行循环查询，直到线性链表结束
                                 * 并根据每个Node的hash值与原数组容量相与得到新的值。
                                 * 与原数组容量相与后的值只会为0 或 原数组容量。
                                 * 根据得到的这两个值 进行判断
                                 * 如果 值为  0             
                                 *    则把他们放到 loHead和loTail指向的新的线性链表当中
                                 *    --尾部插入
                                 * 如果 值为  原数组容量     
                                 *    则把他们放到 hiHead和hiTail指向的新的线性链表当中
                                 *    --尾部插入
                                 */
                                do {
                                            next = e.next;
                                            if ((e.hash & oldCap) == 0) {
                                                    if (loTail == null)
                                                            loHead = e;
                                                    else
                                                        loTail.next = e;
                                                    loTail = e;
                                                }
                                            else {
                                                    if (hiTail == null)
                                                            hiHead = e;
                                                    else
                                                        hiTail.next = e;
                                                    hiTail = e;
                                                }
                                        } while ((e = next) != null);
                                /**
                                 * 原线性链表结束
                                 * 如果新的loTail指向的线性链表不为空，
                                 * 		则把它的最后结尾值的指针指向null值
                                 *  	并把loHeah与loTail指向的新的链表放到新数组
                                 * 		下标为j的位置上。
                                 * 如果新的hiTail指向的线性链表不为空，
                                 * 		则把它的最后结尾值的指针指向null值
                                 *      并把hiHeah与hiTail指向的新的链表放到新数
                                 * 		组下标为 （j + oldCap） 的位置上。
                                 */
                                if (loTail != null) {
                                            loTail.next = null;
                                            newTab[j] = loHead;
                                        }
                                    if (hiTail != null) {
                                            hiTail.next = null;
                                            newTab[j + oldCap] = hiHead;
                                        }
                                }
                        }
                }
        }
    return newTab;
}


```

## TreeNode.split 树结构的节点转移

### 红黑树的概念

[[Map#红黑树]]

与链表扩容类似，就是在遍历完之后要判断原来存储位置和高位存储位置的节点是否小于6，小于就转成链表结构，否则构建新的红黑树；

### 构建红黑树




![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202403052035591.png)



# 参考链接 

[深入理解HashMap原理(一)——HashMap源码解析(JDK 1.8)](https://ost.51cto.com/posts/952)
