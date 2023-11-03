---
author: zjmantou
title: Java集合
time: 2023-10-31 周二
tags:
  - Java
  - 笔记
---
#### 1、集合框架，list，map，set都有哪些具体的实现类，区别都是什么?

Java集合里使用接口来定义功能，是一套完善的继承体系。Iterator是所有集合的总接口，其他所有接口都继承于它，该接口定义了集合的遍历操作，Collection接口继承于Iterator，是集合的次级接口（Map独立存在），定义了集合的一些通用操作。

##### Java集合的类结构图如下所示：

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb0ef8154d864f8dbaf31ea211bc49d1~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

List：有序、可重复；索引查询速度快；插入、删除伴随数据移动，速度慢；

Set：无序，不可重复；

Map：键值对，键唯一，值多个；

1.List,Set都是继承自Collection接口，Map则不是;

2.List特点：元素有放入顺序，元素可重复;

Set特点：元素无放入顺序，元素不可重复，重复元素会盖掉，（注意：元素虽然无放入顺序，但是元素在set中位置是由该元素的HashCode决定的，其位置其实是固定，加入Set 的Object必须定义equals()方法;

另外list支持for循环，也就是通过下标来遍历，也可以使用迭代器，但是set只能用迭代，因为他无序，无法用下标取得想要的值）。

3.Set和List对比：

Set：检索元素效率低下，删除和插入效率高，插入和删除不会引起元素位置改变。

List：和数组类似，List可以动态增长，查找元素效率高，插入删除元素效率低，因为会引起其他元素位置改变。

4.Map适合储存键值对的数据。

5.线程安全集合类与非线程安全集合类

LinkedList、ArrayList、HashSet是非线程安全的，Vector是线程安全的;

HashMap是非线程安全的，HashTable是线程安全的;

StringBuilder是非线程安全的，StringBuffer是线程安的。

##### 下面是这些类具体的使用介绍：

##### ArrayList与LinkedList的区别和适用场景

Arraylist：

优点：ArrayList是实现了基于动态数组的数据结构,因地址连续，一旦数据存储好了，查询操作效率会比较高（在内存里是连着放的）。

缺点：因为地址连续，ArrayList要移动数据,所以插入和删除操作效率比较低。

LinkedList：

优点：LinkedList基于链表的数据结构，地址是任意的，其在开辟内存空间的时候不需要等一个连续的地址，对新增和删除操作add和remove，LinedList比较占优势。LikedList 适用于要头尾操作或插入指定位置的场景。

缺点：因为LinkedList要移动指针,所以查询操作性能比较低。

适用场景分析：

当需要对数据进行对应访问的情况下选用ArrayList，当要对数据进行多次增加删除修改时采用LinkedList。

##### ArrayList和LinkedList怎么动态扩容的吗？

ArrayList:

ArrayList 的初始大小是0，然后，当add第一个元素的时候大小则变成10。并且，在后续扩容的时候会变成当前容量的1.5倍大小。

LinkedList:

linkedList 是一个双向链表，没有初始化大小，也没有扩容的机制，就是一直在前面或者后面新增就好。

##### ArrayList与Vector的区别和适用场景

ArrayList有三个构造方法：

java

复制代码

`public ArrayList(intinitialCapacity)// 构造一个具有指定初始容量的空列表。    public ArrayList()// 构造一个初始容量为10的空列表。 public ArrayList(Collection<? extends E> c)// 构造一个包含指定 collection 的元素的列表`  

Vector有四个构造方法：

java

复制代码

`public Vector() // 使用指定的初始容量和等于零的容量增量构造一个空向量。     public Vector(int initialCapacity) // 构造一个空向量，使其内部数据数组的大小，其标准容量增量为零。     public Vector(Collection<? extends E> c)// 构造一个包含指定 collection 中的元素的向量   public Vector(int initialCapacity, int capacityIncrement)// 使用指定的初始容量和容量增量构造一个空的向量`
   

ArrayList和Vector都是用数组实现的，主要有这么四个区别：

1)Vector是多线程安全的，线程安全就是说多线程访问代码，不会产生不确定的结果。而ArrayList不是，这可以从源码中看出，Vector类中的方法很多有synchronied进行修饰，这样就导致了Vector在效率上无法与ArrayLst相比；

2)两个都是采用的线性连续空间存储元素，但是当空间充足的时候，两个类的增加方式是不同。

3)Vector可以设置增长因子，而ArrayList不可以。

4)Vector是一种老的动态数组，是线程同步的，效率很低，一般不赞成使用。

适用场景：

1.Vector是线程同步的，所以它也是线程安全的，而ArraList是线程异步的，是不安全的。如果不考虑到线程的安全因素，一般用ArrayList效率比较高。

2.如果集合中的元素的数目大于目前集合数组的长度时，在集合中使用数据量比较大的数据，用Vector有一定的优势。

##### HashSet与TreeSet的区别和适用场景

1.TreeSet 是二叉树（红黑树的树据结构）实现的，TreeSet中的数据是自动排好序的，不允许放入null值。

2.HashSet 是哈希表实现的，HashSet中的数据是无序的可以放入null，但只能放入一个null，两者中的值都不重复，就如数据库中唯一约束。

3.HashSet要求放入的对象必须实现HashCode()方法，并且，放入的对象，是以hashcode码作为标识的，而具有相同内容的String对象，hashcode是一样，所以放入的内容不能重复但是同一个类的对象可以放入不同的实例。

适用场景分析:

HashSet是基于Hash算法实现的，其性能通常都优于TreeSet。为快速查找而设计的Set，我们通常都应该使用HashSet，在我们需要排序的功能时，我们才使用TreeSet。

##### HashMap与TreeMap、HashTable的区别及适用场景

HashMap 非线程安全

HashMap：基于哈希表(散列表)实现。使用HashMap要求的键类明确定义了hashCode()和equals()[可以重写hasCode()和equals()]，为了优化HashMap空间的使用，您可以调优初始容量和负载因子。其中散列表的冲突处理主分两种，一种是开放定址法，另一种是链表法。HashMap实现中采用的是链表法。

TreeMap：非线程安全基于红黑树实现。TreeMap没有调优选项，因为该树总处于平衡状态。

适用场景分析：

HashMap和HashTable:HashMap去掉了HashTable的contain方法，但是加上了containsValue()和containsKey()方法。HashTable是同步的，而HashMap是非同步的，效率上比HashTable要高。HashMap允许空键值，而HashTable不允许。

HashMap：适用于Map中插入、删除和定位元素。

Treemap：适用于按自然顺序或自定义顺序遍历键(key)。 (ps:其实我们工作的过程中对集合的使用是很频繁的,稍注意并总结积累一下,在面试的时候应该会回答的很轻松)

#### 2、set集合从原理上如何保证不重复？

1）在往set中添加元素时，如果指定元素不存在，则添加成功。

2）具体来讲：当向HashSet中添加元素的时候，首先计算元素的hashcode值，然后用这个（元素的hashcode）%（HashMap集合的大小）+1计算出这个元素的存储位置，如果这个位置为空，就将元素添加进去；如果不为空，则用equals方法比较元素是否相等，相等就不添加，否则找一个空位添加。

#### 3、HashMap和HashTable的主要区别是什么？，两者底层实现的数据结构是什么？

HashMap和HashTable的区别：

二者都实现了Map 接口，是将唯一的键映射到特定的值上，主要区别在于：

1)HashMap 没有排序，允许一个null 键和多个null 值,而Hashtable 不允许；

2)HashMap 把Hashtable 的contains 方法去掉了，改成containsvalue 和containsKey, 因为contains 方法容易让人引起误解；

3)Hashtable 继承自Dictionary 类，HashMap 是Java1.2 引进的Map 接口的实现；

4)Hashtable 的方法是Synchronized 的，而HashMap 不是，在多个线程访问Hashtable 时，不需要自己为它的方法实现同步，而HashMap 就必须为之提供额外的同步。Hashtable 和HashMap 采用的hash/rehash 算法大致一样，所以性能不会有很大的差异。

HashMap和HashTable的底层实现数据结构：

HashMap和Hashtable的底层实现都是数组 + 链表结构实现的（jdk8以前）

#### 4、HashMap、ConcurrentHashMap、hash()相关原理解析？

##### HashMap 1.7的原理：

HashMap 底层是基于 数组 + 链表 组成的，不过在 jdk1.7 和 1.8 中具体实现稍有不同。

负载因子：

- 给定的默认容量为 16，负载因子为 0.75。Map 在使用过程中不断的往里面存放数据，当数量达到了 16 * 0.75 = 12 就需要将当前 16 的容量进行扩容，而扩容这个过程涉及到 rehash、复制数据等操作，所以非常消耗性能。
- 因此通常建议能提前预估 HashMap 的大小最好，尽量的减少扩容带来的性能损耗。

其实真正存放数据的是 Entry<K,V>[] table，Entry 是 HashMap 中的一个静态内部类，它有key、value、next、hash（key的hashcode）成员变量。

put 方法：

- 判断当前数组是否需要初始化。
- 如果 key 为空，则 put 一个空值进去。
- 根据 key 计算出 hashcode。
- 根据计算出的 hashcode 定位出所在桶。
- 如果桶是一个链表则需要遍历判断里面的 hashcode、key 是否和传入 key 相等，如果相等则进行覆盖，并返回原来的值。
- 如果桶是空的，说明当前位置没有数据存入，新增一个 Entry 对象写入当前位置。（当调用 addEntry 写入 Entry 时需要判断是否需要扩容。如果需要就进行两倍扩充，并将当前的 key 重新 hash 并定位。而在 createEntry 中会将当前位置的桶传入到新建的桶中，如果当前桶有值就会在位置形成链表。）

get 方法：

- 首先也是根据 key 计算出 hashcode，然后定位到具体的桶中。
- 判断该位置是否为链表。
- 不是链表就根据 key、key 的 hashcode 是否相等来返回值。
- 为链表则需要遍历直到 key 及 hashcode 相等时候就返回值。
- 啥都没取到就直接返回 null 。

##### HashMap 1.8的原理：

当 Hash 冲突严重时，在桶上形成的链表会变的越来越长，这样在查询时的效率就会越来越低；时间复杂度为 O(N)，因此 1.8 中重点优化了这个查询效率。

TREEIFY_THRESHOLD 用于判断是否需要将链表转换为红黑树的阈值。

HashEntry 修改为 Node。

put 方法：

- 判断当前桶是否为空，空的就需要初始化（在resize方法 中会判断是否进行初始化）。
- 根据当前 key 的 hashcode 定位到具体的桶中并判断是否为空，为空表明没有 Hash 冲突就直接在当前位置创建一个新桶即可。
- 如果当前桶有值（ Hash 冲突），那么就要比较当前桶中的 key、key 的 hashcode 与写入的 key 是否相等，相等就赋值给 e,在第 8 步的时候会统一进行赋值及返回。
- 如果当前桶为红黑树，那就要按照红黑树的方式写入数据。
- 如果是个链表，就需要将当前的 key、value 封装成一个新节点写入到当前桶的后面（形成链表）。
- 接着判断当前链表的大小是否大于预设的阈值，大于时就要转换为红黑树。
- 如果在遍历过程中找到 key 相同时直接退出遍历。
- 如果 e != null 就相当于存在相同的 key,那就需要将值覆盖。
- 最后判断是否需要进行扩容。

get 方法：

- 首先将 key hash 之后取得所定位的桶。
- 如果桶为空则直接返回 null 。
- 否则判断桶的第一个位置(有可能是链表、红黑树)的 key 是否为查询的 key，是就直接返回 value。
- 如果第一个不匹配，则判断它的下一个是红黑树还是链表。
- 红黑树就按照树的查找方式返回值。
- 不然就按照链表的方式遍历匹配返回值。

修改为红黑树之后查询效率直接提高到了 O(logn)。但是 HashMap 原有的问题也都存在，比如在并发场景下使用时容易出现死循环：

- 在 HashMap 扩容的时候会调用 resize() 方法，就是这里的并发操作容易在一个桶上形成环形链表；这样当获取一个不存在的 key 时，计算出的 index 正好是环形链表的下标就会出现死循环：在 1.7 中 hash 冲突采用的头插法形成的链表，在并发条件下会形成循环链表，一旦有查询落到了这个链表上，当获取不到值时就会死循环。

##### ConcurrentHashMap 1.7原理：

ConcurrentHashMap 采用了分段锁技术，其中 Segment 继承于 ReentrantLock。不会像 HashTable 那样不管是 put 还是 get 操作都需要做同步处理，理论上 ConcurrentHashMap 支持 CurrencyLevel (Segment 数组数量)的线程并发。每当一个线程占用锁访问一个 Segment 时，不会影响到其他的 Segment。

put 方法:

首先是通过 key 定位到 Segment，之后在对应的 Segment 中进行具体的 put。

- 虽然 HashEntry 中的 value 是用 volatile 关键词修饰的，但是并不能保证并发的原子性，所以 put 操作时仍然需要加锁处理。
    
- 首先第一步的时候会尝试获取锁，如果获取失败肯定就有其他线程存在竞争，则利用 scanAndLockForPut() 自旋获取锁:
    
    尝试自旋获取锁。 如果重试的次数达到了 MAX_SCAN_RETRIES 则改为阻塞锁获取，保证能获取成功。
    
- 将当前 Segment 中的 table 通过 key 的 hashcode 定位到 HashEntry。
    
- 遍历该 HashEntry，如果不为空则判断传入的 key 和当前遍历的 key 是否相等，相等则覆盖旧的 value。
    
- 为空则需要新建一个 HashEntry 并加入到 Segment 中，同时会先判断是否需要扩容。
    
- 最后会使用unlock()解除当前 Segment 的锁。
    

get 方法：

- 只需要将 Key 通过 Hash 之后定位到具体的 Segment ，再通过一次 Hash 定位到具体的元素上。
- 由于 HashEntry 中的 value 属性是用 volatile 关键词修饰的，保证了内存可见性，所以每次获取时都是最新值。
- ConcurrentHashMap 的 get 方法是非常高效的，因为整个过程都不需要加锁。

##### ConcurrentHashMap 1.8原理：

1.7 已经解决了并发问题，并且能支持 N 个 Segment 这么多次数的并发，但依然存在 HashMap 在 1.7 版本中的问题：那就是查询遍历链表效率太低。和 1.8 HashMap 结构类似：其中抛弃了原有的 Segment 分段锁，而采用了 CAS + synchronized 来保证并发安全性。

CAS：

如果obj内的value和expect相等，就证明没有其他线程改变过这个变量，那么就更新它为update，如果这一步的CAS没有成功，那就采用自旋的方式继续进行CAS操作。

问题：

- 目前在JDK的atomic包里提供了一个类AtomicStampedReference来解决ABA问题。这个类的compareAndSet方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。
- 如果CAS不成功，则会原地自旋，如果长时间自旋会给CPU带来非常大的执行开销。

put 方法：

- 根据 key 计算出 hashcode 。
- 判断是否需要进行初始化。
- 如果当前 key 定位出的 Node为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋保证成功。
- 如果当前位置的 hashcode == MOVED == -1,则需要进行扩容。
- 如果都不满足，则利用 synchronized 锁写入数据。
- 最后，如果数量大于 TREEIFY_THRESHOLD 则要转换为红黑树。

get 方法：

- 根据计算出来的 hashcode 寻址，如果就在桶上那么直接返回值。
- 如果是红黑树那就按照树的方式获取值。
- 就不满足那就按照链表的方式遍历获取值。

1.8 在 1.7 的数据结构上做了大的改动，采用红黑树之后可以保证查询效率（O(logn)），甚至取消了 ReentrantLock 改为了 synchronized，这样可以看出在新版的 JDK 中对 synchronized 优化是很到位的。

[HashMap、ConcurrentHashMap 1.7/1.8实现原理](https://link.juejin.cn?target=https%3A%2F%2Fcrossoverjie.top%2F2018%2F07%2F23%2Fjava-senior%2FConcurrentHashMap%2F "https://crossoverjie.top/2018/07/23/java-senior/ConcurrentHashMap/")

[hash()算法全解析](https://link.juejin.cn?target=https%3A%2F%2Fwww.hollischuang.com%2Farchives%2F2091 "https://www.hollischuang.com/archives/2091")

##### HashMap何时扩容：

当向容器添加元素的时候，会判断当前容器的元素个数，如果大于等于阈值---即大于当前数组的长度乘以加载因子的值的时候，就要自动扩容。

##### 扩容的算法是什么：

扩容(resize)就是重新计算容量，向HashMap对象里不停的添加元素，而HashMap对象内部的数组无法装载更多的元素时，对象就需要扩大数组的长度，以便能装入更多的元素。当然Java里的数组是无法自动扩容的，方法是使用一个新的数组代替已有的容量小的数组。

##### Hashmap如何解决散列碰撞（必问）？

Java中HashMap是利用“拉链法”处理HashCode的碰撞问题。在调用HashMap的put方法或get方法时，都会首先调用hashcode方法，去查找相关的key，当有冲突时，再调用equals方法。hashMap基于hasing原理，我们通过put和get方法存取对象。当我们将键值对传递给put方法时，他调用键对象的hashCode()方法来计算hashCode，然后找到bucket（哈希桶）位置来存储对象。当获取对象时，通过键对象的equals()方法找到正确的键值对，然后返回值对象。HashMap使用链表来解决碰撞问题，当碰撞发生了，对象将会存储在链表的下一个节点中。hashMap在每个链表节点存储键值对对象。当两个不同的键却有相同的hashCode时，他们会存储在同一个bucket位置的链表中。键对象的equals()来找到键值对。

##### Hashmap底层为什么是线程不安全的？

- 并发场景下使用时容易出现死循环，在 HashMap 扩容的时候会调用 resize() 方法，就是这里的并发操作容易在一个桶上形成环形链表；这样当获取一个不存在的 key 时，计算出的 index 正好是环形链表的下标就会出现死循环；
- 在 1.7 中 hash 冲突采用的头插法形成的链表，在并发条件下会形成循环链表，一旦有查询落到了这个链表上，当获取不到值时就会死循环。

#### 5、ArrayMap跟SparseArray在HashMap上面的改进？

HashMap要存储完这些数据将要不断的扩容，而且在此过程中也需要不断的做hash运算，这将对我们的内存空间造成很大消耗和浪费。

##### SparseArray:

SparseArray比HashMap更省内存，在某些条件下性能更好，主要是因为它避免了对key的自动装箱（int转为Integer类型），它内部则是通过两个数组来进行数据存储的，一个存储key，另外一个存储value，为了优化性能，它内部对数据还采取了压缩的方式来表示稀疏数组的数据，从而节约内存空间，我们从源码中可以看到key和value分别是用数组表示：

ini

复制代码

`private int[] mKeys; private Object[] mValues;`

同时，SparseArray在存储和读取数据时候，使用的是二分查找法。也就是在put添加数据的时候，会使用二分查找法和之前的key比较当前我们添加的元素的key的大小，然后按照从小到大的顺序排列好，所以，SparseArray存储的元素都是按元素的key值从小到大排列好的。 而在获取数据的时候，也是使用二分查找法判断元素的位置，所以，在获取数据的时候非常快，比HashMap快的多。

##### ArrayMap:

ArrayMap利用两个数组，mHashes用来保存每一个key的hash值，mArrray大小为mHashes的2倍，依次保存key和value。

ini

复制代码

`mHashes[index] = hash; mArray[index<<1] = key; mArray[(index<<1)+1] = value;`

当插入时，根据key的hashcode()方法得到hash值，计算出在mArrays的index位置，然后利用二分查找找到对应的位置进行插入，当出现哈希冲突时，会在index的相邻位置插入。

##### 假设数据量都在千级以内的情况下：

1、如果key的类型已经确定为int类型，那么使用SparseArray，因为它避免了自动装箱的过程，如果key为long类型，它还提供了一个LongSparseArray来确保key为long类型时的使用

2、如果key类型为其它的类型，则使用ArrayMap。

### 相关链接
[[Map]]
  

作者：jsonchao  
链接：https://juejin.cn/post/6844904079152381959  
来源：稀土掘金  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。