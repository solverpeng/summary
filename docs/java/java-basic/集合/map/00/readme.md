# HashMap源码解析

在jdk1.7中，HashMap采用数组+链表（拉链法）。

在jdk1.8中，HashMap采用数组+链表+红黑树。

## JDK1.7

### 静态属性

```java
/**
 * 默认初始化容量- 必须是2的整数次幂.
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

/**
 * 2的整数次幂，最大容量不超过2^30次方
 */
static final int MAXIMUM_CAPACITY = 1 << 30;

/**
 * 默认负载因子大小
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;

/**
 * 空表
 */
static final Entry<?,?>[] EMPTY_TABLE = {};
```

### 属性

```java
/**
 * 是一个Entry[]数组类型，而Entry实际上就是一个单向链表。哈希表的"key-value键值对"都是存储在Entry数组中的
 */
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;

/**
 * HashMap的大小，它是HashMap保存的键值对的数量。
 */
transient int size;

/**
 * threshold的值="容量*加载因子"，当HashMap中存储数据的数量达到threshold时，就需要将HashMap的容量加倍
 */
int threshold;

/**
 * 加载因子
 */
final float loadFactor;

/**
 * 实现fail-fast机制，HashMap被修改的次数
 */
transient int modCount;
```

### 链表实现

```java
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    int hash;

    Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    }
}
```

### 构造器

当new HashMap()时，会构造一个初始容量为16，负载因子为0.75的HashMap，且会初始化扩容阙值为16。

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
	// 初始化负载因子
    this.loadFactor = loadFactor;
    // 初始化容量
    threshold = initialCapacity;
    init();
}
```

### put(K key, V value)

```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    addEntry(hash, key, value, i);
    return null;
}

private void inflateTable(int toSize) {
    // 保证容量是2的整数次幂
    int capacity = roundUpToPowerOf2(toSize);
    // 重新计算扩容阙值，默认为容量16*负载因子0.75=12
    threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
    // 将table进行初始化，大小为容量值
    table = new Entry[capacity];
    // 初始化hash种子
    initHashSeedAsNeeded(capacity);
}

// 保证容量是2的整数次幂，并且不超过最大值，若不是2的整数次幂，则取大于当前值且最接近2的整数次幂的数
private static int roundUpToPowerOf2(int number) {
    // Integer.highestOneBit() 最高位为1,其他位为0
    return number >= MAXIMUM_CAPACITY
            ? MAXIMUM_CAPACITY
            : (number > 1) ? Integer.highestOneBit((number - 1) << 1) : 1;
}

private V putForNullKey(V value) {
    // 若table中索引为0已经存在值
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        // 新值替换旧值，返回旧值
        if (e.key == null) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    modCount++;
    // 若table索引为0不存在值，则将对应的entry添加到table的第一个索引处
    addEntry(0, null, value, 0);
    return null;
}

// 计算哈希值
final int hash(Object k) {
    int h = hashSeed;
    if (0 != h && k instanceof String) {
        return sun.misc.Hashing.stringHash32((String) k);
    }

    // 调用key的hashCode()方法得到hashCode值
    h ^= k.hashCode();

    // 此函数确保每个位位置上仅以常量倍数差异的哈希代码具有有限的冲突数（默认加载因子下约为8）。
    // 对hashCode进行一系列的位移与异或运算并把结果作为hashCode返回，为了减少hash冲突，从而提高HashMap的效率
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}

// 计算索引
static int indexFor(int h, int length) {
    // 确定非null key的索引，h & (length-1)与h % length等价，结果不会超过length，因为length是2的整数次幂，所以可以进行位运算，
    // 位运算效率要高于取模运算
    return h & (length-1);
}

void addEntry(int hash, K key, V value, int bucketIndex) {
    // 当容量达到阙值，并产生了hash冲突，进行扩容
    if ((size >= threshold) && (null != table[bucketIndex])) {
        // 容量扩展为之前的2倍
        resize(2 * table.length);
        // 若key为null，则hash值为0
        hash = (null != key) ? hash(key) : 0;
        // 计算索引位置
        bucketIndex = indexFor(hash, table.length);
    }
    // 创建Entry并存储到hash表中
    createEntry(hash, key, value, bucketIndex);
}
```

### 扩容

```java
// 扩容
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }

    // 重新创建底层数组
    Entry[] newTable = new Entry[newCapacity];
    // 对于已经存在的元素，重新计算后放到新的数组中
    transfer(newTable, initHashSeedAsNeeded(newCapacity));
    table = newTable;
    // 更新扩容阙值
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

### get(Object key)

```java
​```java
public V get(Object key) {
	// key为null
    if (key == null)
        return getForNullKey();
    Entry<K,V> entry = getEntry(key);
    // 若entry不为null，则返回该entry的value
    return null == entry ? null : entry.getValue();
}

private V getForNullKey() {
    if (size == 0) {
        return null;
    }
    // 若Entry数组索引为0存在元素且Entry元素的key为null，返回对应的值
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null)
            return e.value;
    }
    // 否则返回null
    return null;
}

final Entry<K,V> getEntry(Object key) {
    if (size == 0) {
        return null;
    }
    // 计算key的hash值，根据hash值和数组长度计算出索引，若对应索引存在元素，且元素key和传入的key相等，则返回该Entry。
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

## JDK1.8

### 静态属性

```java
/**
 * 默认初始化容量-必须是2的整数次幂
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

/**
 * 2的整数次幂，最大容量不超过2^30次方
 */
static final int MAXIMUM_CAPACITY = 1 << 30;

/**
 * 默认负载因子大小
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;

/**
 * 表示桶的树化阀值，大小为8，即单链表转换成红黑树的阀值。当单链表长度大于该值8时才会转换。
 */
static final int TREEIFY_THRESHOLD = 8;

/**
 * 链表还原阀值，表示红黑树转换为单链表结构。
 */
static final int UNTREEIFY_THRESHOLD = 6;

/**
 * 即当哈希表中的容量 > 该值时，才允许树形化链表（即将链表转换成红黑树），否则，若桶内元素太多时，则直接扩容，而不是树形化 // 为了避免进行扩容、树形化选择的冲突，这个值不能小于 4 * 
 * TREEIFY_THRESHOLD。转化红黑树的table容量最小容量64，至少是4*TREEIFY_THRESHOLD（单链表节点个数阀值，用以判断是否需要树形化）=32。用以避免在扩容和树形结构化的阀值 
 * 可能产生的冲突。所以如果小于64必须要扩容。如果在创建HashMap实例时没有给定capacity、loadFactor则默认值分别是16和0.75。
 *  
 * 当好多bin被映射到同一个桶时，如果这个桶中bin的数量小于TREEIFY_THRESHOLD当然不会转化成树形结构存储；如果这个桶中bin的数量大于了 TREEIFY_THRESHOLD ，但是capacity小于MIN_TREEIFY_CAPACITY 
 * 则依然使用单链表结构进行存储，此时会对HashMap进行扩容；如果capacity大于了MIN_TREEIFY_CAPACITY ，则会进行树化。
 */
static final int MIN_TREEIFY_CAPACITY = 64;
```

## 属性

```java
/**
 * 实际存储的数组
 */
transient Node<K,V>[] table;

/**
 * 
 */
transient Set<Map.Entry<K,V>> entrySet;

/**
 * 实际存储的键值对大小
 */
transient int size;

/**
 * 实现fail-fast机制，HashMap被修改的次数
 */
transient int modCount;

/**
 * 扩容阀值，扩容阀值 = 实际容量 × 实际加载因子。
 */
int threshold;

/**
 * 加载因子
 */
final float loadFactor;
```

### 数据结构

链表实现

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
}
```

### 构造器

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}

/**
 * 返回一个比给定整数大且最接近的2的幂次方整数int类型，如给定10，返回2的4次方16。
 */
static final int tableSizeFor(int cap) {
    int n = cap - 1; //如果cap已经是2的幂，又没有执行这个减1操作，则执行完后面的几条无符号右移操作之后，返回的capacity将是这个cap的2倍。
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    // 负数/最大值/正常值+1
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

### put(K key, V value)

> 1. 对key的hashCode()做hash，然后再计算index;
> 2. 如果没碰撞直接放到bucket里；
> 3. 如果碰撞了，以链表的形式存在buckets里；
> 4. 如果碰撞导致链表过长(大于等于TREEIFY_THRESHOLD，默认为8)，就把链表转换成红黑树；
> 5. 如果节点已经存在就替换old value(保证key的唯一性)；
> 6. 如果bucket满了(超过load factor * current capacity)，就进行resize。

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

static final int hash(Object key) {
    int h;
    //先获取键对象的hashCode，即哈希码，然后将该哈希码与该哈希码无符号右移16位相异或，搅动了低16位的hashCode值。
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;

    //如果该键对象的哈希表索引处为null，则表示该位置没有被使用，即直接插入哈希表数组；
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        //如果哈希表索引处的键对象与插入的键对象相等，则直接更新原来的键对象的值；
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        //如果该键对象的索引处的键的类型为红黑树类型，则插入到红黑树；
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        //如果该键对象的索引处的键的类型为单链表类型，则插入到单链表；
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
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
    //更新修改次数modCount
    ++modCount;
    //更新键值对数量size，比较size与threshold大小来判断否超过扩容阀值，若超过，则扩容resize
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

### 扩容

> 1. 无节点，不处理；
> 2. 单节点，重新计算index（hash & (newCap - 1)）。
> 3. 多节点，跟单节同样的情况，只是没有重新计算所有的index，而是看看原来的hash值新增的那个bit是1还是0（因为容量扩大了一倍，因此影响结果的是hash之前没有参与运算的最右侧位值，通过 hash & oldCap 便能得到），是0的话索引没变，是1的话索引变成“原索引+oldCap”。

```java
//扩容函数。在插入键值对时，且发现容量不足，则执行扩容操作。两种情况：一是初始化哈希表；二是发现当前数组容量不足。
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
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

### get(Object key)

> 1. 对key的hashCode()做hash，然后再计算index;
> 2. 如果该 index 下没有值，则直接返回 null；
> 3. index 下有值，判断 table 数组中该下标处第一个节点是否命中，命中则直接返回；
> 4. 未命中，则判断去树结构中查找，或者去链表结构中查找，找到则返回。

```java
public V get(Object key) {
    Node<K,V> e;
    //为null
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // 如果哈希表容量为0或者关键字没有命中，直接返回null
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 通过hash和key判断是否为第一个节点
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            // 以红黑树的方式查找
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {// 遍历链表查找
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

## 对比

### hash()方法

在 JDK1.7中，使用了4次位运算+5次异或。

```java
h ^= k.hashCode();
h ^= (h >>> 20) ^ (h >>> 12);
h ^ (h >>> 7) ^ (h >>> 4);
```

在JDK1.8中，1次位运算+1次异或。

```java
(h = key.hashCode()) ^ (h >>> 16);
```

### 索引计算

JDK1.7，```h & (length-1);```

JDK1.8，```(n - 1) & hash;```

### 扩容

JDK1.7中，元素个数大于等于阈值且产生哈希碰撞的时候，进行扩容

```java
if ((size >= threshold) && (null != table[bucketIndex])) {
    // 容量扩展为之前的2倍
    resize(2 * table.length);
    // 若key为null，则hash值为0
    hash = (null != key) ? hash(key) : 0;
    // 计算索引位置
    bucketIndex = indexFor(hash, table.length);
}
```

JDK1.8中，

```java
if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
    newThr = oldThr << 1; // double threshold
}
```



## 不同点

1. JDK1.7用的是头插法，而JDK1.8及之后使用的都是尾插法，那么他们为什么要这样做呢？因为JDK1.7是用单链表进行的纵向延伸，当采用头插法时会容易出现逆序且环形链表死循环问题。但是在JDK1.8之后是因为加入了红黑树使用尾插法，能够避免出现逆序且链表死循环的问题。
2. 扩容后数据存储位置的计算方式也不一样：1. 在JDK1.7的时候是直接用hash值和需要扩容的二进制数进行&（这里就是为什么扩容的时候为啥一定必须是2的多少次幂的原因所在，因为如果只有2的n次幂的情况时最后一位二进制数才一定是1，这样能最大程度减少hash碰撞）（hash值 & length-1）
3. 而在JDK1.8的时候直接用了JDK1.7的时候计算的规律，也就是扩容前的原始位置+扩容的大小值=JDK1.8的计算方式，而不再是JDK1.7的那种异或的方法。但是这种方式就相当于只需要判断Hash值的新增参与运算的位是0还是1就直接迅速计算出了扩容后的储存方式。
4. JDK1.7的时候使用的是数组+ 单链表的数据结构。但是在JDK1.8及之后时，使用的是数组+链表+红黑树的数据结构（当链表的深度达到8的时候，也就是默认阈值，就会自动扩容把链表转成红黑树的数据结构来把时间复杂度从O（n）变成O（logN）提高了效率）



## 问题

1. 哈希表如何解决Hash冲突？
   - 预防措施
     - 好的Hash算法
       - hashCode()
       - 扰动处理
       - 数组长度要求为2的次幂
     - 扩容机制
       - 哈希表大于等于扩容阈值时，就会扩容哈希表
       - 扩容阈值=扩容因子 * 容量
   - 解决方案
     - 数据结构
       - jdk1.7=数组+链表
       - jdk1.8=数组+链表+红黑树
     - 数据存储机制
       - jdk1.7=发生冲突时，采用链地址法+头插法
       - jdk1.8=发生冲突时，采用 链地址法+尾插法+红黑树

2. 为什么键值允许为null？
   - 键=唯一&可以为null
     - key为null时，hash值默认为0，只能又一个key为null
     - value不需要唯一，可以为多个null
3. 为什么是线程不安全的？
   - jdk1.7
     - 无同步锁
     - 多线程下容易出现resize()死循环
       - 并发执行put操作导致触发扩容行为，从而导致环形链表
       - 在获取数据遍历链表时形成死循环
   - jdk1.8
     - 无同步锁
   - fail-fast策略
     - 在使用迭代器的过程中出现并发操作（并发修改），抛出 ConcurrentModificationException异常
     - 原理
       - 变量modCount表示修改次数，没修过1次HashMap内容，都将增加改变量值
       - 在迭代器初始化时，将该值付给迭代器expectedModCount变量
       - 判断modCount是否与expectedModCount相等
         - 相等，则表示无其他线程修改
         - 不等，则表示又其他线程修改
4. 为什么不保证有序性？
   1. 插入顺序与存储顺序不同
      - 插入顺序=用户操作的顺序
      - 存储顺序=根据hash算法计算而来的数组下标顺序
5. 为什么位置会随时间变化？
   - 存在扩容操作从而导致存储位置重新计算，从而发生变化
6. 为什么HashMap中String、Integer这样的包装类适合作为key键
   - String、Integer等包装类的特性，保证了 Hash 值得不可更改性 & 计算准确性，减少了发生hash碰撞得几率
   - String、Integer包装类的特性
     - final类型
       - 具有不可变性
       - 保证了key的不可更改性，不会出现放入&获取时哈希码不同的情况
       - 内部重写了equals()和hashCode()方法