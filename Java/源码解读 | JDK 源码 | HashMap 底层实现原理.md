[TOC]

[TOC]

## 前言

HashMap 可能是我们平时开发中用到的最多的数据结构，也是面试中面试官最喜欢考察的一个知识点，今天我们就分析一下 HashMap 的底层实现原理。

HashMap 在 JDK1.8 版本做了一些改进，数据结构的存储由**数组+链表**的方式，变化为**数组+链表+红黑树**的存储方式，性能方面也做了很多优化。所以我们首先分析 1.7 版本的实现原理，这样能减少理解的难度。理解了 1.7 版本的原理后再来看 1.8 做了哪些修改。

### 其他知识点准备

解读源码之前，请确保已经掌握以下基础知识

* 了解 Java二进制位运算&、|、~、^，移位运算>>、<<、>>> 
* 理解 Java 中的 ==，equals 与 hashCode 的区别与联系
* 了解基本的数据结构——数组、链表、红黑树

### Map 类图结构

![image-20190426150448868](https://kolly-imgstore.oss-cn-shenzhen.aliyuncs.com/img/Java-Map类图结构.png)

### 相关概念

#### 哈希的概念

Hash 就是把任意长度的输入(又叫做预映射， pre-image)，通过哈希算法，变换成固定长度的输出(通常是整型)，该输出就是哈希值。这种转换是一种 **压缩映射** ，也就是说，散列值的空间通常远小于输入的空间。**不同的输入可能会散列成相同的输出，从而不可能从散列值来唯一的确定输入值。简单的说，就是一种将任意长度的消息压缩到某一固定长度的息摘要函数。**

#### 哈希的应用：哈希表

我们知道，数组的特点是：寻址容易，插入和删除困难；而链表的特点是：寻址困难，插入和删除容易。那么我们能不能综合两者的特性，做出一种**寻址容易，插入和删除也容易的数据结构**呢？答案是肯定的，这就是**哈希表**。



## 1.7 版本原理解读

### HashMap 的数据结构

我们知道，在 Java 中最常用的两种结构是 **数组** 和 **链表**，几乎所有的数据结构都可以利用这两种来组合实现，HashMap 就是这种应用的一个典型。实际上，HashMap 就是一个 **链表数组**(JDK1.8用链表+红黑树替代链表)，如下是它数据结构：

![image-20190426152710698](https://kolly-imgstore.oss-cn-shenzhen.aliyuncs.com/img/HashMap%20结构-1.7.png)

### HashMap 的快速存取



### HashMap 源码解读

#### 类签名

```java
public class HashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
```

#### 基本属性

```java
    // 初始容量大小设置为 16(数组初始化的长度)
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    // 最大容量大小设置为 2 的 32 次方
    static final int MAXIMUM_CAPACITY = 1 << 30;

    // 默认负载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    static final Entry<?,?>[] EMPTY_TABLE = {};

		// 存放元素的数组对象，元素个数达到临界值会扩容，任何时候数组长度必须为 2 的 次方
    transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;

    // 哈希表中的元素个数
    transient int size;

    // 元素个数达到此临界值会触发扩容 (threshold = capacity * load factor).
    // If table == EMPTY_TABLE then this is the initial capacity at which the
    // table will be created when inflated.
    int threshold;

    // 负载因子
    final float loadFactor;

    // HashMap 结构被修改的次数(元素数量变化或者扩容)
    // 用于快速失败 (See ConcurrentModificationException)
    transient int modCount;
```

#### 内部类Entry

从上面 table 属性可以看出，HashMap 底层数组存放的数据类型是一个内部类 `Entry`，他实现了`Entry`接口并在内部保留了一个对自身类型的引用`next`，这也是其能形成链表的基础。

```java
static class Entry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        Entry<K,V> next;
        int hash;

        /**
         * Creates new entry.
         */
        Entry(int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }
    }
```

#### 构造函数

```java
    // 构造一个指定初始容量和指定负载因子的空 HashMap
    public HashMap(int initialCapacity, float loadFactor) {
        this.loadFactor = loadFactor;
        threshold = initialCapacity; 
    }

    // 构造一个指定初始容量和默认负载因子 (0.75)的空 HashMap
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    // 构造一个默认初始容量（16）和默认负载因子 (0.75）的空 HashMap
    public HashMap() {
        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
    }
```

#### 关键方法解析：put 方法

```java
public V put(K key, V value) {
  			// 如果 table 为空，初始化 table
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
  			// 当 key 为 null 时，调用 putForNullKey 方法，并将该键值对保存到 table 的第一个位置
        if (key == null)
            return putForNullKey(value);
  
  			// 根据 key 的 hashcode 重新计算 hash 值
        int hash = hash(key);
  			// 根据新的 hash 值计算数据应该放在数组的哪个位置(哪个桶)
        int i = indexFor(hash, table.length);
  
  			// 判断该位置上是否已经有元素，如果有，遍历该位置上的链表，判断 key 是否存在，如果存在则替换
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++; // 结构被修改的次数加 1
        addEntry(hash, key, value, i); // 将key-vlaue, 生成Entry实体，添加到HashMap中的Entry[]数组中
        return null;
    }

private V putForNullKey(V value) {
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {
            if (e.key == null) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        modCount++;
        addEntry(0, null, value, 0);
        return null;
    }

void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length); // 扩容为原来的两倍
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }

        createEntry(hash, key, value, bucketIndex);
    }

void createEntry(int hash, K key, V value, int bucketIndex) {
        Entry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new Entry<>(hash, key, value, e);
        size++;
    }
```

* 从`addEntry`方法的实现可以看出，当 HashMap 中元素的个数达到`threshold`临界值之后，table 会**扩容两倍**
* 从`putForNullKey`方法的实现可以看出，HashMap 中可以保存键为NULL的键值对，它会被放在 table 的第一个元素的位置，也就是索引为 0 的位置

##### hash 方法

```java
final int hash(Object k) {
        int h = hashSeed;
        if (0 != h && k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }

        h ^= k.hashCode();

        // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
        // number of collisions (approximately 8 at default load factor).
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }
```

* 使用hash()方法对一个对象的hashCode进行重新计算是为了防止质量低下的hashCode()函数实现。由于hashMap的支撑数组长度总是 2 的幂次，通过右移可以使低位的数据尽量的不同，从而使hash值的分布尽量均匀

##### indexFor方法 

```java
static int indexFor(int h, int length) {
        // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
        return h & (length-1);
    }
```

* 当length为2的n次方时，`h & (length - 1)`就相当于对length取模，而且速度比直接取模要快得多

#### 关键方法解析：get 方法

```java
public V get(Object key) {
        if (key == null)
            return getForNullKey();
        Entry<K,V> entry = getEntry(key);

        return null == entry ? null : entry.getValue();
    }

private V getForNullKey() {
        if (size == 0) {
            return null;
        }
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {
            if (e.key == null)
                return e.value;
        }
        return null;
    }

final Entry<K,V> getEntry(Object key) {
        if (size == 0) {
            return null;
        }

        int hash = (key == null) ? 0 : hash(key);
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k))))
                return e;
        }
        return null;
    }
```

#### HashMap 的底层数组长度为何总是2的n次方

```java
static int indexFor(int h, int length) {
        // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
        return h & (length-1);
    }
```

* 为了使数据在桶中均匀分布减少碰撞，采用取模的算法，也就是 hash % length，根据取模的结果决定放到哪个桶中
* 计算机中直接求余效率不如位移运算，如果length是2的n次方，那么hash & (length-1)就等同于hash % length，但是效率更高
* 有兴趣的朋友可以写一个 main 方法验证一下直接【求余】和【按位】运算的耗时差别，效率还是差不少的

#### 扩容方法：resize

```java
void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        Entry[] newTable = new Entry[newCapacity];
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        table = newTable;
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }

void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        for (Entry<K,V> e : table) {
            while(null != e) {
                Entry<K,V> next = e.next;
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            }
        }
    }
```

##### HashMap 扩容在多线程情况下出现死循环的现象

在并发的情况，发生扩容时，可能会产生`循环链表`，在执行`get`的时候，会触发`死循环`，引起CPU的100%问题，所以一定要避免在并发环境下使用HashMap

**在1.8之前，新插入的元素都是放在了链表的头部位置，但是这种操作在高并发的环境下容易导致死锁，所以1.8之后，新插入的元素都放在了链表的尾部**

## JDK 1.8的 改变

在Jdk1.8中HashMap的实现方式做了一些改变，但是基本思想还是没有变得，只是在一些地方做了优化，下面来看一下这些改变的地方，数据结构的存储由**数组+链表**的方式，变化为**数组+链表+红黑树**的存储方式，在性能上进一步得到提升。

#### 数据的存储方式

![image-20190426143045554](https://kolly-imgstore.oss-cn-shenzhen.aliyuncs.com/img/HashMap-JDK1.8版本.png)

#### put 方法解析

```java
public V put(K key, V value) {
    //调用putVal()方法完成
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //判断table是否初始化，否则初始化操作
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //计算存储的索引位置，如果没有元素，直接赋值
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        //节点若已经存在，执行赋值操作
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        //判断链表是否是红黑树
        else if (p instanceof TreeNode)
            //红黑树对象操作
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //为链表，
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    //链表长度8，将链表转化为红黑树存储
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                //key存在，直接覆盖
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    //记录修改次数
    ++modCount;
    //判断是否需要扩容
    if (++size > threshold)
        resize();
    //空操作
    afterNodeInsertion(evict);
    return null;
}
```



## 写在后面

### 总结

* HashMap 扩容在多线程情况下出现死循环，导致CPU100%现象，主要原因是多个线程执行扩容操作时可能导致链表形成环，在 get 的时候出现死循环，1.8 版本已经解决(新节点放在链表尾部)

### 参考资料