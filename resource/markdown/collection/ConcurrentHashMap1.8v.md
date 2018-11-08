> :dog:本文是基于 `jdk1.8.0_151` 分析
>

> 如果你已经阅读了我之前写的关于 `HashMap` 和 `ConcurrentHashMap` [文章](https://github.com/about-cloud/JavaCore)，对本篇 `jdk1.8版` 的`ConcurrentHashMap` 源码分析更容量理解。

:family:

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、ConcurrentHashMap的扩展关系</h3>

```java
public class ConcurrentHashMap<K, V> extends AbstractMap<K, V>
        implements ConcurrentMap<K, V>, Serializable
```

> 从扩展关系上，没有任何变化，但 `ConcurrentMap<K, V>` 又添加几个 `default` 方法，`default` 的好处在[上篇文章](https://github.com/about-cloud/JavaCore)中已经提到过：

```java
/**
 * forEach 迭代方法
 *
 * @throws NullPointerException {@inheritDoc}
 * @since 1.8
 */
default void forEach(BiConsumer<? super K, ? super V> action) {
    Objects.requireNonNull(action);
    for (Map.Entry<K, V> entry : entrySet()) {
        K k;
        V v;
        try {
            k = entry.getKey();
            v = entry.getValue();
        } catch(IllegalStateException ise) {
            continue;
        }
        action.accept(k, v);
    }
}
/** 其他略 */
```



⛄️⛄️⛄️

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、ConcurrentHashMap数据结构</h3>

####回顾 jdk 1.7 的 ConcurrentHashMap 的数据结构

> `jdk 1.7` 的`ConcurrentHashMap`是基于 **分段锁** 机制设计的，将一个大的Map分割成n个小的 **段segment**，对每段进行加锁，降低了容器加锁的粒子度，每段(segment)各自加锁，互不影响，当一个线程访问 Map 其中一段数据时，其他段的数据也能被线程正常访问。分段锁使用的锁是 `ReentrantLock` 可重入锁。

![ConcurrentHashMap1.7v](http://pgq1yfr0p.bkt.clouddn.com/image/java/collection/segments.png)

#### :star2:优化后的 jdk 1.8 的 ConcurrentHashMap 的数据结构

> `jdk 1.8` 的 `ConcurrentHashMap` 

TODO



#### 🌟重要的字段

```java
/** 最大容量 10.7亿+ */
private static final int MAXIMUM_CAPACITY = 1 << 30;
/** table的默认初始容量 16，容量必须为 2 的次幂 */
private static final int DEFAULT_CAPACITY = 16;



```





#### :star2:重要的字段

```java
/** table的默认初始容量 16，可以通过构造函数指定初始容量 */
static final int DEFAULT_INITIAL_CAPACITY = 16;
/** table的默认负载因子 0.75，可以通过构造函数指定负载因子大小 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;
/** table 默认并发等级 16，可以通过构造函数指定并发等级 */
static final int DEFAULT_CONCURRENCY_LEVEL = 16;
/** 最大容量 10.7亿+ */
static final int MAXIMUM_CAPACITY = 1 << 30;
/** 每个段表的最小容量。容量必须是 2的次幂，
  * 容量最少是2，以避免在延迟构造后，为了下次使用而又立即调整大小。
  */
static final int MIN_SEGMENT_TABLE_CAPACITY = 2;
/** 允许的最大段数；可以是使用构造方法的蚕食指定。但必须是2的次幂，并且小于1＜<24。 */
static final int MAX_SEGMENTS = 1 << 16; // 略微保守的段数
/** 加锁失败时的重试次数 */
static final int RETRIES_BEFORE_LOCK = 2;
/** 分割段时使用的掩码值。用来对segment进行定位，判断应该落在哪个段segment中 */
final int segmentMask;
/** 段内索引的偏移量 */
final int segmentShift;
/** 将原来整个大的哈希表分割成n个小的哈希表，这里的每段就是专用的小的哈希表。 */
final Segment<K,V>[] segments;
```

#### :briefcase:ConcurrentHashMap 列表条目

> 它的元素节点项 `HashEntry<K,V>` 是内部独有静态类，不像 `HashMap` 的 `Entry<K, V>` 实现至 `Map.Entry<K, V>`。它们都很相似，但不同的是 节点`HashEntry<K,V>`被 `final` 修饰表示被会被继承，在 `HashEntry` 静态内部类的内部 key 和 hash 是被 `final` 修饰，赋予其不能被修改的特性。 value和指向下一个节点的变量next是被 `volatile` 修饰的，表示具有 **可见性** ，所以 **读操作** 在无需加锁的情况下总能读取最新的数据 。

```java
static final class HashEntry<K,V> {
    // 具有不可更改特性的 hash 和 key
    final int hash;
    final K key;
    // 具有可见性的 value 和 next
    volatile V value;
    volatile HashEntry<K,V> next;

    HashEntry(int hash, K key, V value, HashEntry<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    /** 设置具有 volatile 特性的 next 属性值 */
    final void setNext(HashEntry<K,V> n) {
        // objectFieldOffset 这个方法的意思是获取字段属性的偏移量（也就是内存位置）
        UNSAFE.putOrderedObject(this, nextOffset, n);
    }

    // Unsafe 操作
    ...
}
```

> :tropical_drink:这里介绍一下 `Unsafe` 类，它位于 `sun.misc.` 包下，具有像 **C语言** 中指针一样的功能，能大大提升效率，也能够操作内存空间，更会存在像 指针 那样的问题。它不属于标准Java类库，可以看成第三方开发工具库。Oracle原计划从Java 9中去掉Unsafe类，现在我看了下 `Java 11` 它已经被移到 `jdk.internal.misc` 这个包下（jdk8时还在sun.misc包中 ）。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、segment段</h3>

> `ConcurrentHashMap` 的主干就是`Segment<K,V> segments`，每个 `segment` 又含有 `HashEntry<K, V> table` ，你可以宽泛的认为每段就是一个 HashEntry 数组，就像 HashMap 的 `Entry<K, V> table` 一样，但粒子度不在一个 level 上。

**分段(segment)** 的设计是 `ConcurrentHashMap` 所特有的功能。`Segment<K,V>` 类恰逢其时的继承了 `ReentrantLock` 重入锁的特性。作为 `ReentrantLock` 的子类，只是为了简化一些 **锁:lock:** 的设计，避免分开创建其他锁。

```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {
    private static final long serialVersionUID = 2249069246763182397L;

    /** 当获取锁失败时，重试次数。与Java虚拟机可用的CPU处理器的个数有关 */
    static final int MAX_SCAN_RETRIES =
        Runtime.getRuntime().availableProcessors() > 1 ? 64 : 1;
        // Runtime.getRuntime().availableProcessors() 获取Java虚拟机可用的处理器数量

    /**
     * 每段被volatile修饰的 table散列表（实际存放元素的地方）。
     * 通过 entryAt/setEntryAt 方法访问元素
     */
    transient volatile HashEntry<K,V>[] table;

    /** 元素的个数。仅在加锁或者保持volatile可见性的情况下读取（访问）*/
    transient int count;

    /** 此段的被更改操作的总次数 */
    transient int modCount;

    /** 阈值（容量 * 负载因子） */
    transient int threshold;

    /** 负载因子 */
    final float loadFactor;

    Segment(float lf, int threshold, HashEntry<K,V>[] tab) {
        this.loadFactor = lf;
        this.threshold = threshold;
        this.table = tab;
    }
...
    /**
     * Scans for a node containing given key while trying to
     * acquire lock, creating and returning one if not found. Upon
     * return, guarantees that lock is held. UNlike in most
     * methods, calls to method equals are not screened: Since
     * traversal speed doesn't matter, we might as well help warm
     * up the associated code and accesses as well.
     *
     * @return a new node if key not found, else null
     */
    private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
        HashEntry<K,V> first = entryForHash(this, hash);
        HashEntry<K,V> e = first;
        HashEntry<K,V> node = null;
        int retries = -1; // negative while locating node
        while (!tryLock()) {
            HashEntry<K,V> f; // to recheck first below
            if (retries < 0) {
                if (e == null) {
                    if (node == null) // speculatively create node
                        node = new HashEntry<K,V>(hash, key, value, null);
                    retries = 0;
                }
                else if (key.equals(e.key))
                    retries = 0;
                else
                    e = e.next;
            }
            else if (++retries > MAX_SCAN_RETRIES) {
                lock();
                break;
            }
            else if ((retries & 1) == 0 &&
                     (f = entryForHash(this, hash)) != first) {
                e = first = f; // re-traverse if entry changed
                retries = -1;
            }
        }
        return node;
    }

    /**
     * Scans for a node containing the given key while trying to
     * acquire lock for a remove or replace operation. Upon
     * return, guarantees that lock is held.  Note that we must
     * lock even if the key is not found, to ensure sequential
     * consistency of updates.
     */
    private void scanAndLock(Object key, int hash) {
        // similar to but simpler than scanAndLockForPut
        HashEntry<K,V> first = entryForHash(this, hash);
        HashEntry<K,V> e = first;
        int retries = -1;
        while (!tryLock()) {
            HashEntry<K,V> f;
            if (retries < 0) {
                if (e == null || key.equals(e.key))
                    retries = 0;
                else
                    e = e.next;
            }
            else if (++retries > MAX_SCAN_RETRIES) {
                lock();
                break;
            }
            else if ((retries & 1) == 0 &&
                     (f = entryForHash(this, hash)) != first) {
                e = first = f;
                retries = -1;
            }
        }
    }

    /**
     * Remove; match on key only if value null, else match both.
     */
    final V remove(Object key, int hash, Object value) {
        if (!tryLock())
            scanAndLock(key, hash);
        V oldValue = null;
        try {
            HashEntry<K,V>[] tab = table;
            int index = (tab.length - 1) & hash;
            HashEntry<K,V> e = entryAt(tab, index);
            HashEntry<K,V> pred = null;
            while (e != null) {
                K k;
                HashEntry<K,V> next = e.next;
                if ((k = e.key) == key ||
                    (e.hash == hash && key.equals(k))) {
                    V v = e.value;
                    if (value == null || value == v || value.equals(v)) {
                        if (pred == null)
                            setEntryAt(tab, index, next);
                        else
                            pred.setNext(next);
                        ++modCount;
                        --count;
                        oldValue = v;
                    }
                    break;
                }
                pred = e;
                e = next;
            }
        } finally {
            unlock();
        }
        return oldValue;
    }

    final boolean replace(K key, int hash, V oldValue, V newValue) {
        if (!tryLock())
            scanAndLock(key, hash);
        boolean replaced = false;
        try {
            HashEntry<K,V> e;
            for (e = entryForHash(this, hash); e != null; e = e.next) {
                K k;
                if ((k = e.key) == key ||
                    (e.hash == hash && key.equals(k))) {
                    if (oldValue.equals(e.value)) {
                        e.value = newValue;
                        ++modCount;
                        replaced = true;
                    }
                    break;
                }
            }
        } finally {
            unlock();
        }
        return replaced;
    }

    final V replace(K key, int hash, V value) {
        if (!tryLock())
            scanAndLock(key, hash);
        V oldValue = null;
        try {
            HashEntry<K,V> e;
            for (e = entryForHash(this, hash); e != null; e = e.next) {
                K k;
                if ((k = e.key) == key ||
                    (e.hash == hash && key.equals(k))) {
                    oldValue = e.value;
                    e.value = value;
                    ++modCount;
                    break;
                }
            }
        } finally {
            unlock();
        }
        return oldValue;
    }

    final void clear() {
        lock();
        try {
            HashEntry<K,V>[] tab = table;
            for (int i = 0; i < tab.length ; i++)
                setEntryAt(tab, i, null);
            ++modCount;
            count = 0;
        } finally {
            unlock();
        }
    }
}
```



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、构造方法</h3>

#### ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel)

```java
/**
 * @param initialCapacity  初始化容量
 * @param loadFactor       负载因子
 * @param concurrencyLevel 并发等级
 */
public ConcurrentHashMap(int initialCapacity,
                         float loadFactor, int concurrencyLevel) {
    // 负载因子必须大于0，初始化容量不能小于0，并发等级必须大于0
    // 否则就会抛出非法参数异常
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    // 保证并发等级最大为 1 << 16
    if (concurrencyLevel > MAX_SEGMENTS)
        concurrencyLevel = MAX_SEGMENTS;
    // 下面这些操作用户计算 segments 长度、具体要分成多少段
    int sshift = 0;
    int ssize = 1;
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;
    }
    this.segmentShift = 32 - sshift;
    this.segmentMask = ssize - 1;
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    int c = initialCapacity / ssize;
    if (c * ssize < initialCapacity)
        ++c;
    int cap = MIN_SEGMENT_TABLE_CAPACITY;
    while (cap < c)
        cap <<= 1;
    // 创建单个segment用于填充segments[0]位置(因为实际存储元素是在HashEntry上)
    Segment<K,V> s0 =
        new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                         (HashEntry<K,V>[])new HashEntry[cap]);
    // 创建 segments数组
    Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
    // 将 s0段填充到 segments[0]
    UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
    // 设置 segments
    this.segments = ss;
}
```



:flashlight::flashlight::flashlight:下面的这两个构造方法调用的是上面这个构造方法，如果指定参数就使用指定的参数，否则使用默认参数。

#### ConcurrentHashMap(int initialCapacity)

```java
public ConcurrentHashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR, DEFAULT_CONCURRENCY_LEVEL);
}
```

#### ConcurrentHashMap()

```java
public ConcurrentHashMap() {
        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR, DEFAULT_CONCURRENCY_LEVEL);
    }
```

> 另一个构造方法这里就不分析了



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">四、添加方法</h3>

> 通过上面的分析，我们知道 segments数组本身不是用来存放 **元素** 的，它是用来存储 `HashEntry<K, V>[] tab 数组`的， 真正存储元素的是 `HashEntry<K, V>[] tab`。下面我们通过源码来分析元素是如何存储的：

#### 添加键值对的 put 方法:heavy_plus_sign:

```java
public V put(K key, V value) {
    Segment<K,V> s;
    // 不允许 value 为 null 值，否则抛出NPE
    if (value == null)
        throw new NullPointerException();
    // 计算key的哈希码
    int hash = hash(key);
    // 计算应该落到哪一段segment(段号)
    int j = (hash >>> segmentShift) & segmentMask;
    // 如果该片段为null，那么进入ensureSegment(int k)方法处理
    if ((s = (Segment<K,V>)UNSAFE.getObject(segments, (j << SSHIFT) + SBASE)) == null)
        s = ensureSegment(j);
    // 添加并返回value
    return s.put(key, hash, value, false);
}
```

#### 确保分段不为null的方法:heavy_check_mark:

```java
/**
 * 返回指定的index处的segment段。如果不存在，就在 segments数组中（通过CAS自旋）创建并记录
 *
 * @param k 指定段的索引
 * @return 此段segment
 */
@SuppressWarnings("unchecked")
private Segment<K,V> ensureSegment(int k) {
    // segment集合数组
    final Segment<K,V>[] ss = this.segments;
    long u = (k << SSHIFT) + SBASE; // 原始偏移量
    Segment<K,V> seg;
    // 获取 segments（这里是ss）集合数组在偏移量u位置的那一段
    // 如果此段为null，那么就创建
    if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
        // 使用 segments[0]这一段作为原型
        Segment<K,V> proto = ss[0];
        // cap指的是 HashEntry<K, V> table数组的容量(capacity)，也就是数组的长度
        int cap = proto.table.length;
        // 负载因子
        float lf = proto.loadFactor;
        // 阈值 = 容量 * 负载因子
        int threshold = (int)(cap * lf);
        // 有了必要的参数后，就开始构造实际存放元素的HashEntry数组了
        HashEntry<K,V>[] tab = (HashEntry<K,V>[])new HashEntry[cap];
        // 再次判断想要的segment是否存在
        if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
            == null) {
            // 如果还未null那么久创建一个segment(并把构造号的HashEntry数组放入)
            Segment<K,V> s = new Segment<K,V>(lf, threshold, tab);
            // 通过CAS自旋抢占资源方式来确保刚构造的segment这一段放入 ss(既segments)中
            while ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
                   == null) {
                if (UNSAFE.compareAndSwapObject(ss, u, null, seg = s))
                    break;
            }
        }
    }
    // 返回
    return seg;
}
```

#### :boat::boat::boat:实际将键值对添加到集合中的方法

```java
// 这里很像HashMap添加元素的操作
// (可参考往期文章：https://github.com/about-cloud/JavaCore)
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    // 先尝试获取锁，如果成功获取那么返回🔙null
    // 如果加锁失败，那么就通过scanAndLockForPut方法扫描加锁来存放元素(详见下面👇分析)
    HashEntry<K,V> node = tryLock() ? null :
        scanAndLockForPut(key, hash, value);
    V oldValue;
    try {
        // 此段中存放元素的哈希表
        HashEntry<K,V>[] tab = table;
        // 通过 逻辑与&🌧 计算出桶号(哈希槽位置)(请参考HashMap)
        int index = (tab.length - 1) & hash;
        // 根据桶号获取桶顶的元素
        HashEntry<K,V> first = entryAt(tab, index);
        // 遍历链表
        for (HashEntry<K,V> e = first;;) {
            // 元素不为空的情况下
            if (e != null) {
                K k;
                // 通过比较 key 或者 key的哈希码和key来判断key是否相等
                // 如果 key 相等，那么替换value
                if ((k = e.key) == key || (e.hash == hash && key.equals(k))) {
                    oldValue = e.value;
                    if (!onlyIfAbsent) {
                        e.value = value;
                        ++modCount;
                    }
                    break;
                }
                // 指定的key与此元素的key不相等，那么就移到下一个元素
                e = e.next;
            }
            // 迭代到当前元素的为null情况
            else {
                // 通过scanAndLockForPut得到方法放入的元素
                if (node != null)
                    // 使用UNSAFE.putOrderedObject方法以“下沉”的方式，
                    // 将元素链入链表，新的元素在桶顶，旧的元素在下面
                    node.setNext(first);
                // 如果 node == null，以为着通过tryLock()获得了锁🔐
                else
                    // 构造新🆕元素
                    node = new HashEntry<K,V>(hash, key, value, first);
                // 判断新添加此元素后，是否超过容量阈值
                int c = count + 1;
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    // 如果超过了容器的负载量，那么进行扩容
                    rehash(node);
                else
                    // 如果没有容量够用，那么就在指定的位置，
                    // 间接UNSAFE.putOrderedObject方法通过添加➕元素
                    setEntryAt(tab, index, node);
                ++modCount;
                // 记录📝实际存放元素的大小
                count = c;
                oldValue = null; // 释放旧值的引用，让GC去处理吧
                break;
            }
        }
    } finally {
        // 🔚解锁
        unlock();
    }
    return oldValue;
}
```

#### （获取锁失败后，线程就会进入等待状态）此时通过自定义扫描、加锁的方式来存放元素

```java
private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
    // 通过当前段segment和指定key的哈希码来获取元素
    HashEntry<K,V> first = entryForHash(this, hash);
    HashEntry<K,V> e = first;
    HashEntry<K,V> node = null;
    // 重试次数
    int retries = -1;
    // 如果加锁失败，就循环
    while (!tryLock()) {
        HashEntry<K,V> f; // to recheck first below
        // 重试次数小于0表示重来都没有创建过元素
        if (retries < 0) {
            // 指定哈希码定位的位置存在null元素，表示此处没有元素，下面就创建元素
            if (e == null) {
                // 如果node再为null的化，就大胆的创建元素吧
                if (node == null)
                    node = new HashEntry<K,V>(hash, key, value, null);
                // 赋值0，表示首次操作
                retries = 0;
            }
            // 指定哈希码定位的位置有非空元素，那就比较key是否相同
            else if (key.equals(e.key))
                // 既然非空，就意味着创建过
                retries = 0;
            else
                // 遍历链表中下一个元素
                e = e.next;
        }
        // 当达到最大重试次数时，加锁跳出循环
        else if (++retries > MAX_SCAN_RETRIES) {
            lock();
            break;
        }
        // 在重试次数允许范围内
        // (retries & 1) == 0 为true，表示retries为偶数(请注意上面做了++retries操作)
        else if ((retries & 1) == 0 &&
                 // 为true，意味着节点被更改了
                 (f = entryForHash(this, hash)) != first) {
            // 节点被更改了，那就替换芯节点
            e = first = f;
            // 再重新开始
            retries = -1;
        }
    }
    return node;
}
```

#### 私有的扩容方法

```java
/**
 * table（指的是HashEntrH<K, V>[]）2倍扩容，
 * 然后在新的table重新排放所有的元素
 * (你看看，扩容多麻烦、又消耗性能，所以初始时一定要合理的指定初始容量)
 * 并且将给定的节点添加到散列表
 */
@SuppressWarnings("unchecked")
private void rehash(HashEntry<K,V> node) {
    // 当前的散列表(哈希是音译，我觉得“散列”更能表达其真实含义)
    HashEntry<K,V>[] oldTable = table;
    // 当前散列表的长度(即容量)
    int oldCapacity = oldTable.length;
    // 新的容量(2倍于当前散列表的容量)
    int newCapacity = oldCapacity << 1;
    // 新阈值
    threshold = (int)(newCapacity * loadFactor);
    // 新的散列表
    HashEntry<K,V>[] newTable =
        (HashEntry<K,V>[]) new HashEntry[newCapacity];
    // 掩码(数组的最大索引值)
    int sizeMask = newCapacity - 1;
    // 迭代数组（散列表），将当前散列表中的元素转移到新的、扩容的散列表中
    for (int i = 0; i < oldCapacity ; i++) {
        HashEntry<K,V> e = oldTable[i];
        if (e != null) {
            // 如果哈希槽处(桶顶)的元素不为空，就准备遍历这个链表
            // 提前获取下一个节点的引用
            HashEntry<K,V> next = e.next;
            // 计算出该元素在新散列表中哈希槽的位置(在新数组中的索引)
            int idx = e.hash & sizeMask;
            // 桶顶处的下一个元素为空，意味着这是个单元素的链表，
            // 直接将元素移到新散列表中
            if (next == null)
                newTable[idx] = e;
            // 否则就意为着链表中有多个元素
            else {
                 // 转移链表中元素的思路，就是遍历这个链表，将元素一个一个的转移
                HashEntry<K,V> lastRun = e; // 记录📝链表中的最后一个元素
                int lastIdx = idx; // 记录📝链表中的最后一个元素应该落在新散列表中的索引
                // 这里的for循环不是遍历、转移元素，而是获取链表中最后一个元素的信息
                for (HashEntry<K,V> last = next;
                     last != null;
                     last = last.next) {
                    // 计算出该元素在新散列表中哈希槽的位置(在新数组中的索引)
                    // 直到遍历链表中的最后一个元素，那么也就获取到链表中的最后一个元素啦
                    int k = last.hash & sizeMask;
                    if (k != lastIdx) {
                        lastIdx = k;
                        lastRun = last;
                    }
                }
                // 把链表中的元素转移到新链表中
                newTable[lastIdx] = lastRun;
                // 外层for循环的一开始已经把链表中第一个元素(桶顶的元素)转移走了，
                // 下面就转移接续的元素
                // 克隆剩下的元素节点
                for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
                    V v = p.value;
                    int h = p.hash;
                    int k = h & sizeMask;
                    HashEntry<K,V> n = newTable[k];
                    // 从外面看，这里是深复制(上面这个链表中的最后一个节点是浅复制)
                    newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
                }
            }
        }
    }
    
    // 扩容的也扩容了，所有的元素也都转移了
    // 这才开始添加用户要添加的新节点
    int nodeIndex = node.hash & sizeMask;
    // 借助UNSAFE.putOrderedObject方法来实现节点的添加
    node.setNext(newTable[nodeIndex]);
    newTable[nodeIndex] = node;
    table = newTable;
}
```

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">五、删除方法</h3>

> 删除元素的思路：定位到 **段segment** ，segment 是用来存放 HashEntry散列表的，散列表table是实际存放元素的地方，然后再定位散列表的位置，判断桶顶是否有元素，如果有的话再遍历链表。删除元素的最复杂的操作是删除链中的元素（注意解链、再接链就可以了），可以参考这里关于LinkedList源码分析更详细的文章：
> https://github.com/about-cloud/JavaCore
> 当然后面还会持续更新本文，有兴趣可以关注上面GitHub文章。