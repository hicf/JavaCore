> :dog:本文是基于 `jdk1.7.0_79` 分析
>
> 本文内容较多，我删减后篇幅还是较长，长期有耐心，:stew:慢慢解读吧。

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、非线程安全HashMap</h3>

> 前面[文章](https://github.com/about-cloud/JavaCore)分析了 `HashMap` 源码，但其操作是非现在安全的，比如两个线程并发赋值，其中key相同，而value不相同，就有可能造成值覆盖的情况。再比如一个线程并发删操作、另一个线性并发写操作，也可能造成空转问题。`java.util.concurrent` 包 `ConcurrentHashMap` 是其线程安全的实现。



:family:

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、ConcurrentHashMap的扩展关系</h3>

```java
public class ConcurrentHashMap<K, V> extends AbstractMap<K, V>
        implements ConcurrentMap<K, V>, Serializable
```

> 抽象类 `AbstractMap` 实现了一些常用的方法，接口 `ConcurrentMap` 也是继承至 `Map` 接口，它具有并发操作的支持。
>
> 接口 `ConcurrentMap` 定义的未实现方法如下：

```java
// 添加不存在的元素
V putIfAbsent(K key, V value);
// 删除元素
boolean remove(Object key, Object value);
// 替换指定key的value，替换成功返回true，否则返回false
boolean replace(K key, V oldValue, V newValue);
// 替换指定key的value
V replace(K key, V value);
```



⛄️⛄️⛄️

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、ConcurrentHashMap数据结构</h3>

> `ConcurrentHashMap`是基于 **分段锁** 机制设计的，将一个大的Map分割成n个小的 **段segment**，对每段进行加锁，降低了容器加锁的粒子度，每段(segment)各自加锁，互不影响，当一个线程访问 Map 其中一段数据时，其他段的数据也能被线程正常访问。分段锁使用的锁是 `ReentrantLock` 可重入锁。

![ConcurrentHashMap1.7v]()

#### 常量

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
```

#### 字段(部分含义请结合下图)

```java
 /* ----------------  -------------- */
/** 用于分割成段的掩码值。密钥的哈希代码的高位位用于选择该段。 */
用来对segment进行定位，判断哪个segment
segment的偏移，segment中的索引
final int segmentMask;
/** 段内索引的移位值。 */
final int segmentShift;
/** 将原来整个大的哈希表分割成n个小的哈希表，这里的每段就是专用的小的哈希表。 */
final Segment<K,V>[] segments;
```

![字段]()

#### :briefcase:ConcurrentHashMap 列表条目

> 它的元素节点项 `HashEntry<K,V>` 是内部独有静态类，不像 `HashMap` 的 Entry<K, V> 实现至 Map.Entry<K, V>。它们都很相似，但不同的是 节点HashEntry<K,V>被 `final` 修饰表示被会被继承，在 `HashEntry` 静态内部类的内部 key 和 hash 是被 `final` 修饰，赋予其不能被修改的特性。 value和指向下一个节点的变量next是被 `volatile` 修饰的，表示具有 **可见性** ，所以 **读操作** 在无需加锁的情况下总能读取最新的数据 。

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
        UNSAFE.putOrderedObject(this, nextOffset, n);
    }

    // Unsafe 操作
    static final sun.misc.Unsafe UNSAFE;
    static final long nextOffset;
    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class k = HashEntry.class;
            // objectFieldOffset 这个方法的意思是获取字段属性的偏移量（也就是内存位置）
            nextOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("next"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
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

    /** 每段被volatile修饰的 table散列表。通过 entryAt/setEntryAt 方法访问元素 */
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

    final V put(K key, int hash, V value, boolean onlyIfAbsent) {
        HashEntry<K,V> node = tryLock() ? null :
            scanAndLockForPut(key, hash, value);
        V oldValue;
        try {
            HashEntry<K,V>[] tab = table;
            int index = (tab.length - 1) & hash;
            HashEntry<K,V> first = entryAt(tab, index);
            for (HashEntry<K,V> e = first;;) {
                if (e != null) {
                    K k;
                    if ((k = e.key) == key ||
                        (e.hash == hash && key.equals(k))) {
                        oldValue = e.value;
                        if (!onlyIfAbsent) {
                            e.value = value;
                            ++modCount;
                        }
                        break;
                    }
                    e = e.next;
                }
                else {
                    if (node != null)
                        node.setNext(first);
                    else
                        node = new HashEntry<K,V>(hash, key, value, first);
                    int c = count + 1;
                    if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                        rehash(node);
                    else
                        setEntryAt(tab, index, node);
                    ++modCount;
                    count = c;
                    oldValue = null;
                    break;
                }
            }
        } finally {
            unlock();
        }
        return oldValue;
    }

    /**
     * Doubles size of table and repacks entries, also adding the
     * given node to new table
     */
    @SuppressWarnings("unchecked")
    private void rehash(HashEntry<K,V> node) {
        /*
         * Reclassify nodes in each list to new table.  Because we
         * are using power-of-two expansion, the elements from
         * each bin must either stay at same index, or move with a
         * power of two offset. We eliminate unnecessary node
         * creation by catching cases where old nodes can be
         * reused because their next fields won't change.
         * Statistically, at the default threshold, only about
         * one-sixth of them need cloning when a table
         * doubles. The nodes they replace will be garbage
         * collectable as soon as they are no longer referenced by
         * any reader thread that may be in the midst of
         * concurrently traversing table. Entry accesses use plain
         * array indexing because they are followed by volatile
         * table write.
         */
        HashEntry<K,V>[] oldTable = table;
        int oldCapacity = oldTable.length;
        int newCapacity = oldCapacity << 1;
        threshold = (int)(newCapacity * loadFactor);
        HashEntry<K,V>[] newTable =
            (HashEntry<K,V>[]) new HashEntry[newCapacity];
        int sizeMask = newCapacity - 1;
        for (int i = 0; i < oldCapacity ; i++) {
            HashEntry<K,V> e = oldTable[i];
            if (e != null) {
                HashEntry<K,V> next = e.next;
                int idx = e.hash & sizeMask;
                if (next == null)   //  Single node on list
                    newTable[idx] = e;
                else { // Reuse consecutive sequence at same slot
                    HashEntry<K,V> lastRun = e;
                    int lastIdx = idx;
                    for (HashEntry<K,V> last = next;
                         last != null;
                         last = last.next) {
                        int k = last.hash & sizeMask;
                        if (k != lastIdx) {
                            lastIdx = k;
                            lastRun = last;
                        }
                    }
                    newTable[lastIdx] = lastRun;
                    // Clone remaining nodes
                    for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
                        V v = p.value;
                        int h = p.hash;
                        int k = h & sizeMask;
                        HashEntry<K,V> n = newTable[k];
                        newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
                    }
                }
            }
        }
        int nodeIndex = node.hash & sizeMask; // add the new node
        node.setNext(newTable[nodeIndex]);
        newTable[nodeIndex] = node;
        table = newTable;
    }

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
    // 查找 2的次幂 个最匹配的参数
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
    // 创建 segments数组 and segments[0]
    Segment<K,V> s0 =
        new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                         (HashEntry<K,V>[])new HashEntry[cap]);
    Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
    UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
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

> 另一个构造方法就不讲解了



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">四、添加方法</h3>



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">五、删除方法</h3>