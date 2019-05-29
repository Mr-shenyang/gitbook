# 1 Hash链表结构
在看HashMap源码之前，先复习下Hash表结构：
> 散列表（Hash table，也叫哈希表），是根据键（Key）而直接访问在内存存储位置的数据结构。也就是说，它通过计算一个关于键值的函数，将所需查询的数据映射到表中一个位置来访问记录，这加快了查找速度。这个映射函数称做散列函数，存放记录的数组称做散列表。
> 
> * 若关键字为 ***k*** ，则其值存放在 ***f(k)*** 的存储位置上。由此，不需比较便可直接取得所查记录。称这个对应关系f为散列函数，按这个思想建立的表为散列表。
> 
> * 对不同的关键字可能得到同一散列地址，即 ***k1 != k2***，而 ***f(k1)=f(k2)*** ，这种现象称为冲突（英语：Collision）。具有相同函数值的关键字对该散列函数来说称做同义词。综上所述，根据散列函数f(k)和处理冲突的方法将一组关键字映射到一个有限的连续的地址集（区间）上，并以关键字在地址集中的“像”作为记录在表中的存储位置，这种表便称为散列表，这一映射过程称为散列造表或散列，所得的存储位置称散列地址。
> 
> * 若对于关键字集合中的任一个关键字，经散列函数映象到地址集合中任何一个地址的概率是相等的，则称此类散列函数为 ***均匀散列函数（Uniform Hash function）*** ， 这就是使关键字经过散列函数得到一个“随机的地址”，从而减少冲突。

维基百科中关于Hash链表的描述清晰的定义了哈希结构的关键因素key、散列函数、冲突。其图示结构如下
![image](/images/map_hash_structure.jpg)

# 2 HashMap结构简述
HashMap继承自Map，实现了Map的主结构Entry<K,V>，包含key、value以及指向下一个Entry的引用
```java
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    /***
     * 对key进行hashcode运算后得到的hash值，存储在Entry，避免重复计算
     */
    int hash;

    Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    }
}
```

定义好主要结构后，HashMap中还定义了

```java
public class HashMap<K, V> {
    /**
     * HashMap的主干数组
     */
    transient HashMap.Entry<K, V>[] table;
    /**
     * 实际存储的key-value键值对的个数
     */
    transient int size;
    /**
     * 阈值
     */
    int threshold;
    /**
     * 负载因子，代表了table的填充度有多少，默认是0.75
     */
    final float loadFactor;
    /**
     * 用于快速失败
     */
    transient int modCount;
}
```
# 3 HashMap详解
## 3.1 关于初始化
```java
public class HashMap<K, V> {
    //......
    public HashMap(){
        this(16, 0.75F);
    }
    public HashMap(int initialCapacity){
        this(initialCapacity, 0.75F);
    }
    public HashMap(int initialCapacity, float loadFactor){
        //一堆校验后调用
        this.init();
    }

    public HashMap(Map<? extends K, ? extends V> map){
        this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
                    DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);
        inflateTable(threshold);
        putAllForCreate(m);
    }

    void init() {
    }
}
```
jdk1.7中为HashMap提供了4个构造函数，其中三个方法，一堆参数初始化后殊途同归的走向init()方法，当我满心欢喜以为马上就要揭开核心时发现，我操什么鬼，init里面什么都没有。注释都没有。只能心里默默的念叨，好吧看来初始化就是纯粹意义上的初始化。
不过三个构造函数一通看完并不是没有收获，比如
* HashMap  默认容量是16
* HashMap  （loadFactor）默认负载因子是0.75
* HashMap  （threshold）初始阈值是容量

## 3.2 关于写操作
我们知道jdk1.7的Map规范中定义了，Map的写操作就两个，put(K key,V value)和putAll(Map<? extends K, ? extends V> map)

这里先看下put操作
```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        /**
         * 如果当前的table没有被分配block则inflate table
         */
        inflateTable(threshold);
    }
    /**
     * 划重点，key可以为空
     */
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    /**
     * 如果该对应数据已存在，执行覆盖操作。用新value替换旧value，并返回旧value
     */
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

/**
 * 核心是初始化物理存储位置
 */
private void inflateTable(int toSize) {
    /**
     * Find a power of 2 >= toSize 意思是说找到一个大于等于toSize的2的幂次
     */
    int capacity = roundUpToPowerOf2(toSize);
    /**
     * 此处为threshold赋值，取capacity*loadFactor和MAXIMUM_CAPACITY+1的最小值，capaticy一定不会超过MAXIMUM_CAPACITY，除非loadFactor大于1
     */
    threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
    table = new Entry[capacity];
    initHashSeedAsNeeded(capacity);
}

private static int roundUpToPowerOf2(int number) {
    // 保证number在[1,MAXIMUM_CAPACITY]区间
    return number >= MAXIMUM_CAPACITY
            ? MAXIMUM_CAPACITY
            : (number > 1) ? Integer.highestOneBit((number - 1) << 1) : 1;
}
``` 
1. 先聊聊方法inflateTable：该方法先是通过Integer的highestOneBit方法对toSize进行取2的幂次处理，然后重新计算阈值、初始化一个大小为capacity的数组。

2. 然后我们在看看如果key是null的情况：如果key=null 则将value放入table[0]的位置或者table[0]的冲突链上
```
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
```
3. 关于hash
简单说来就是如果没有设置hashSeed，则对key的hashcode进一步进行计算以及二进制位的调整等来保证最终获取的存储位置尽量分布均匀
```java
final int hash(Object k) {
        int h = hashSeed;
        if (0 != h && k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }

        h ^= k.hashCode();
        /**
         * 对k.hashCode()进一步调整，最终使尽可能均匀分布
         */
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }
```

4. 关于indexFor，进一步处理来获取实际的存储位置
```java
static int indexFor(int h, int length) {
    return h & (length-1);
}
```
inflateTable中我们说到table的capacity是2的幂次，所以h于上length-1就是如下所示了

比如h=18，length=16
```
        1  0  0  1  0
    &   0  1  1  1  1
    __________________
        0  0  0  1  0    = 2
```
所以存储位置是table[2]或者其对于的冲突处理链上；

5. 关于添加键值对addEntry

```java
void addEntry(int hash, K key, V value, int bucketIndex) {
    /***
     * size>=当前阀值，并且对于的table[bucketIndex]位置不为空，则扩容为当前table.length的两倍
     * 关于扩容暂且跳过，等会细聊
     */
    if ((size >= threshold) && (null != table[bucketIndex])) {
        resize(2 * table.length);
        //扩容后重新计算hash
        hash = (null != key) ? hash(key) : 0;
        //扩容后重新计算bucketIndex
        bucketIndex = indexFor(hash, table.length);
    }
    //创建一个新的条目
    createEntry(hash, key, value, bucketIndex);
}

/**
 * 创建一个新的Entry，并且Entry中的next指向当前table[bucketIndex]上的值，将新创建的条目作为当前table[bucketIndex]
 */
void createEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}
```
至此put方法完了。

最后说下总体put的总体流程

```flow
st=>start: 键值对
bolckIsNull=>condition: 是否分配空间
keyIsnNull=>condition: key is null?
valueIsExist=>condition: value is exist?
overThreshold=>condition: 超过阈值且主干不为空

allocationBolck=>operation: 分配空间
hash=>operation: 获取hash编码
indexFor=>operation: 获取主干位置(bucketIndex)
processNullKey=>operation: hash=0,bucketIndex=0
processExistKey=>operation: 用新的value替换旧value
returnExistValue=>operation: 返回旧value
createEntry=>operation: 添加数据
returnNull=>operation: 返回null
resize=>operation: 扩容
reHash=>operation: 重新hash
reIndexFor=>operation: 重获主干位置(bucketIndex)
e=>end: End


st->bolckIsNull
bolckIsNull(yes)->keyIsnNull
bolckIsNull(no,right)->allocationBolck->keyIsnNull
keyIsnNull(yes,right)->processNullKey->valueIsExist
keyIsnNull(no)->hash->indexFor->valueIsExist
valueIsExist(yes)->processExistKey->returnExistValue->e
valueIsExist(no)->overThreshold
overThreshold(yes,right)->resize
resize(right)->reHash(right)->reIndexFor->createEntry
overThreshold(no)->createEntry
createEntry->returnNull->e

```
## 3.3 关于扩容
在进行put操作是有个判定，如果size>=当前阀值，并且对于的table[bucketIndex]位置不为空就调用resize方法进行扩容。
```java
    /**
     * 扩容容量是当前table.length的两倍
     */
    void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }
        //新建一个容量为newCapacity的bucket
        Entry[] newTable = new Entry[newCapacity];
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        table = newTable;
        //计算新的阀值
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }

    /**
     * 将table上的数据转移到newTable上
     */
    void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        //循环table桶上所有链条上的条目，重新hash、indexFor
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
## 3.4 关于读操作

```java
    public V get(Object key) {
        if (key == null)
            return getForNullKey();
        Entry<K,V> entry = getEntry(key);

        return null == entry ? null : entry.getValue();
    }
    /**
     * 获取key为null的值
     */
    private V getForNullKey() {
        /**
         * 避免table没有被初始化造成数组越界问题
         */
        if (size == 0) {
            return null;
        }
        /**
         * 在table[0]上寻找value
         */
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {
            if (e.key == null)
                return e.value;
        }
        return null;
    }

    final Entry<K,V> getEntry(Object key) {
        /**
         * 避免table没有被初始化造成数组越界问题
         */
        if (size == 0) {
            return null;
        }
        /**
         * 计算key的hash值
         */
        int hash = (key == null) ? 0 : hash(key);
        /**
         * bucketIndex = indexFor(hash, table.length)
         * 循环遍历table[bucketIndex]上的键值对
         */
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            /**
             * 如果键值对的key和传入key的hash值相同并且equals
             * 返回对应的值
             */
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k))))
                return e;
        }
        return null;
    }
```
通过代码阅读，可以看到get方法比较简单。
```flow
st=>start: key
sizeIsZero=>condition: tableSize is 0
keyIsNull=>condition: key is null
nullKeyValue=>operation: table[0]链表中找null对应的value
findkeyValue=>operation: table[bucketIndex]链表中找key对应的value
hash=>operation: 计算key的hash
indexFor=>operation: 计算key对应的存储位置
returnNull=>operation: 返回空
existKey=>condition: 存在对应的key
returnValue=>operation: 返回值
e=>end: end

st->sizeIsZero(no)->keyIsNull
sizeIsZero(yes,right)->returnNull
keyIsNull(yes,right)->nullKeyValue->existKey
keyIsNull(no)->hash->indexFor->findkeyValue->existKey
existKey(yes)->returnValue->e
existKey(no)->returnNull->e
```
## 3.5 关于移除操作
```java
/***
 * 移除hashmap中指定的key
 */
public V remove(Object key) {
    Entry<K,V> e = removeEntryForKey(key);
    return (e == null ? null : e.value);
}

/**
 * 寻找Entry，并移除
 */
final Entry<K,V> removeEntryForKey(Object key) {
    if (size == 0) {
        return null;
    }
    /**
     * 计算hash
     */
    int hash = (key == null) ? 0 : hash(key);
    /**
     * 计算主干位置
     */
    int i = indexFor(hash, table.length);
    Entry<K,V> prev = table[i];
    Entry<K,V> e = prev;
    /**
     * 移除链表上的Entry
     */
    while (e != null) {
        Entry<K,V> next = e.next;
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k)))) {
            modCount++;
            size--;
            if (prev == e)
                table[i] = next;
            else
                prev.next = next;
            e.recordRemoval(this);
            return e;
        }
        prev = e;
        e = next;
    }
    return e;
}
```
可以看到，hashmap的移除操作相对还是很简单的

# 4 jdk1.8新特性
读完了1.7的HashMap源码我以为终于掌握了一套葵花宝典，发现我想多了。1.8中HashMap对于Hash碰撞的解决方案做了比较大的改动，引入了**红黑树**来解决碰撞。闲话少叙，直接上源码吧。
## 4.1 结构变化
1.7中的hashmap底层结构就一个Entry，到了1.8时变成了两个，一个是Node，一个是TreeNode，需要注意下，TreeNode继承自LinkedHashMap.Entry。原因暂且不表
```java
/***
 * 树状节点
 */
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
}
/**
 * 链表结构
 */
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
}
```
## 4.2 写操作变化

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //table如果没有被初始化，则进行初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //如果没有发生碰撞，则直接赋值
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        //如果冲突位置上的key和传入的key一样则新的节点
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        //如果冲突位置上的节点是红黑树节点，到红黑树种执行节点插入
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        //如果冲突位置上的节点是红黑树节点，插入红黑树中
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    //如果链表的长度大于8是则将链表转换成红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        //如果节点的key在链表中已存在时，e不为空。需要进行value替换
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```