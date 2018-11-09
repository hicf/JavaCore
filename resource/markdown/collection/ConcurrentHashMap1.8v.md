> :relaxed:本文是基于 `jdk1.8.0_151` 分析
>

> 如果你已经阅读了我之前写的[关于 **jdk1.8 HashMap** 和  **jdk1.7 ConcurrentHashMap** 文章](https://github.com/about-cloud/JavaCore)，对于本篇 `jdk1.8版` 的`ConcurrentHashMap` 源码分析更容量理解，不过没关系，本篇文章也非常详细地来分析。

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

#### 回顾 jdk 1.7 的 ConcurrentHashMap 的数据结构

> `jdk 1.7` 的`ConcurrentHashMap`是基于 **分段锁** 机制设计的，将一个大的Map分割成n个小的 **段segment**，对每段进行加锁，降低了容器加锁的粒子度，每段(segment)各自加锁，互不影响，当一个线程访问 Map 其中一段数据时，其他段的数据也能被线程正常访问。分段锁使用的锁是 `ReentrantLock` 可重入锁。

![ConcurrentHashMap1.7v](http://pgq1yfr0p.bkt.clouddn.com/image/java/collection/segments.png)

#### :star2:优化后的 jdk 1.8 的 ConcurrentHashMap 的数据结构

> `jdk 1.8` 的 `ConcurrentHashMap`  相对于较前的版本该动还是蛮大。
>
> 首先取消了 `分段锁` 的设计，而改用像 `jdk 1.8` 中 `HashMap` 那样的数据结构：**数组** + **链表** + **红黑树**。
>
> 再次，在保证线程安全的问题了取消了 `ReentrantLock` 改用 `CAS` + `synchronized` 保证并发的安全。（当然ReentrantLock 的原理也是基于CAS的），在使用 `synchronized` 时并没有像 `Hashtable` 那样粗暴地锁住整个数组，而它是锁住单个节点。TODO

![jdk1.8 ConcurrentHashMap ]()



#### 🌟重要的字段

```java
/** 最大容量 10.7亿+ */
private static final int MAXIMUM_CAPACITY = 1 << 30;
/** table的默认初始容量 16，容量必须为 2 的次幂 */
private static final int DEFAULT_CAPACITY = 16;
/** table 默认并发等级 16 */
private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
/** table的默认负载因子 0.75，可以通过构造函数指定负载因子大小 */
private static final float LOAD_FACTOR = 0.75f;
/** （从链表上的元素个数来讲）链表转红黑树的阈值 */
static final int TREEIFY_THRESHOLD = 8;
/** （从红黑树上的元素个数来讲）红黑树转链表的阈值 */
static final int UNTREEIFY_THRESHOLD = 6;
/** 链表转红黑树的另一个阈值(条件)：Map的容量至少为64. */
static final int MIN_TREEIFY_CAPACITY = 64;
/**  每次进行转移的最小值 */
private static final int MIN_TRANSFER_STRIDE = 16;
/** 生成sizeCtl所使用的bit位数 */
private static int RESIZE_STAMP_BITS = 16;
/** 进行扩容所允许的最大线程数 */
private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;
/** 可以有效使用的 CPU 数量 */
static final int NCPU = Runtime.getRuntime().availableProcessors();

/** 首先哈希值不小于0，下面的几个常量相当于标识位 */
/** 用于转换节点的哈希值 */
static final int MOVED     = -1;
/** 用于红黑树根节点的哈希码的标识位 */
static final int TREEBIN   = -2;
/** 容器中还有一个保留节点，此处也是有关的哈希码的标识位 */
static final int RESERVED  = -3;
/** 用于普通节点哈希码的标识位 */
static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash

/** 容器的数组（哈希表、散列表），在第一次插入元素时才初始化，大小总是2的次幂 */
transient volatile Node<K,V>[] table;
/** 扩容时使用的临时过度的数组（仅使用于扩容） */
private transient volatile Node<K,V>[] nextTable;
/** 容器中实际存储元素的数量。(利用CAS自旋来更新其值) */
private transient volatile long baseCount;
/**
 * 不同的值起到不同的作用，但唯一功能是起到控制作用，你可以认为它是控制标识符
 * sizeCtl小于0 标识容器正在初始化或扩容
 * sizeCtl为-1 标识正式初始化
 * sizeCtl为-N 标识有N-1个线程正在进行扩容操作
 */
private transient volatile int sizeCtl;
/** 调整表大小时使用，下一个transfer任务的起始下标index 加上1 之后的值 */
private transient volatile int transferIndex;
/** 当初始化或者扩容时，CAS自旋锁的标识位 */
private transient volatile int cellsBusy;
/** 计数器池，非空时，大小为2的次幂 */
private transient volatile CounterCell[] counterCells;
```



#### :sun_with_face: 大多数用于存储元素的 Node<K, V> 节点（链表必用的节点）

> `ConcurrentHashMap` 重点元素 **项HashEntry<K, V>** 在 `jdk1.8` 已改为 **节点Node<K, V>**，与 `jdk1.7`版本的自定义略有不同，`jdk1.8` 中是实现于 `Map.Entry` 接口的。 下面就来分析一下 `jdk1.8`  的`ConcurrentHashMap` **Node<K, V>** 节点：
>
> 🔞请注意：这个**Node<K, V>** 是**大多数** 用于存储的元素节点，并不是全部，而**红黑树** 是用下面的 **TreeNode<K, V>** 节点作为元素存储节点。因为 **链表** 中的节点只有一个后继节点，而 **TreeNode<K, V>** 作为二叉树中的节点，最多可有两个后继节点（既左、右子节点）。
>
> 可参考[往期的文章](https://github.com/about-cloud/JavaCore)：https://github.com/about-cloud/JavaCore

```java
/** 实现于 Map.Entry */
static class Node<K,V> implements Map.Entry<K,V> {
    // // final 修饰的 哈希码、key，防止被重复赋值
    final int hash;
    final K key;
    // 具有可见性的 val 和 next
    volatile V val;
    // 当前节点指向的下一个节点
    volatile Node<K,V> next;

    /**
     * 构造方法用于注入 node节点 的属性值(或引用)
     * 参数从左至右依次是：key的哈希码，key，value，指向的下一个节点next
     */
    Node(int hash, K key, V val, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.val = val;
        this.next = next;
    }
	// getter & toString 方法
    public final K getKey()       { return key; }
    public final V getValue()     { return val; }
    public final String toString(){ return key + "=" + val; }
    // 返回节点的哈希码
    public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
    // 设置 value 的方法，可能会抛出不支持的操作异常
    public final V setValue(V value) {
        throw new UnsupportedOperationException();
    }

    // 用于节点比较是否相等的方法
    public final boolean equals(Object o) {
        Object k, v, u; Map.Entry<?,?> e;
        // 返回判断key、value是否相同的结果
        return ((o instanceof Map.Entry) &&
                (k = (e = (Map.Entry<?,?>)o).getKey()) != null &&
                (v = e.getValue()) != null &&
                (k == key || k.equals(key)) &&
                (v == (u = val) || v.equals(u)));
    }

    /**
     * 虚拟化地支持 map.get() 操作; 子类可以重写此方法.
     */
    Node<K,V> find(int h, Object k) {
        Node<K,V> e = this;
        if (k != null) {
            // 循环遍历链表
            do {
                K ek;
                if (e.hash == h &&
                    ((ek = e.key) == k || (ek != null && k.equals(ek))))
                    return e;
            } while ((e = e.next) != null);
        }
        return null;
    }
}
```



#### :tada:专用于红黑树的 TreeNode<K, V> 节点

```java
static final class TreeNode<K,V> extends Node<K,V> {
    // 父节点
    TreeNode<K,V> parent;
    // 左子节点
    TreeNode<K,V> left;
    // 右子节点
    TreeNode<K,V> right;
    // 指向上一个节点（一般是父节点），删除节点时会用到
    TreeNode<K,V> prev;
    // 红黑标识：true表示此节点为红色，false表示此节点为黑色
    boolean red;
	// 有参构造方法
    TreeNode(int hash, K key, V val, Node<K,V> next,
             TreeNode<K,V> parent) {
        super(hash, key, val, next);
        this.parent = parent;
    }

    Node<K,V> find(int h, Object k) {
        return findTreeNode(h, k, null);
    }

    /**
     * 查找并返回红黑树中是存在的节点，不存在返回null
     * 在上篇关于jdk1.8HashMap源码分析的文章中，分析过类似的（写）操作
     * 文章持续更新地址：https://github.com/about-cloud/JavaCore
     */
    final TreeNode<K,V> findTreeNode(int h, Object k, Class<?> kc) {
        if (k != null) {
            // 当前节点
            TreeNode<K,V> p = this;
            // 循环遍历平衡树
            do  {
                // ph：当前节点的哈希码，
                // dir：搜索的方向，左或右：-1表示左，1表示右，
                // pk：当前节点的key
                // q：当前节点
                int ph, dir; K pk; TreeNode<K,V> q;
                // pl：当前节点p的左子节点；pr：当前节点p的右子节点
                TreeNode<K,V> pl = p.left, pr = p.right;
                if ((ph = p.hash) > h)
                    // 当前节点的哈希值大于指定的哈希值，指向左子节点
                    p = pl;
                else if (ph < h)
                    // 当前节点的哈希值小于指定的哈希值，指向右子节点
                    p = pr;
                else if ((pk = p.key) == k || (pk != null && k.equals(pk)))
                    // 如果当前节点的key与指定的k相同，那么就直接返回此节点
                    return p;
                else if (pl == null)
                    // 如果左子节点为空，就向右子节点查找
                    p = pr;
                else if (pr == null)
                    // 如果右子节点为空，就向左子节点查找
                    p = pl;
                // 判断 k 的类是否实现了比较器
                else if ((kc != null ||
                          // 判断 k 的类是否实现了比较器
                          (kc = comparableClassFor(k)) != null) &&
                         // 这里实际是 pk == null || pk.getClass() != kc ? 0 :
                         // ((Comparable)pk).compareTo(pk)
                         // 下面是解读这个三目运算：
                         // pk == null 表示判断当前节点是否为null
                         // pk.getClass() != kc 表示当前节点对象的类和key的对象的类是否不同
                         // ((Comparable)k).compareTo(pk)表示将指定的key与当前节点的key比较
                         (dir = compareComparables(kc, k, pk)) != 0)
                    // dir小于0表示向左子节点搜索
                    p = (dir < 0) ? pl : pr;
                // 循环查找
                else if ((q = pr.findTreeNode(h, k, kc)) != null)
                    return q;
                else
                    p = pl;
            } while (p != null);
        }
        return null;
    }
}
```



:bus: computeIfAbsent 和 compute 方法使用的位置保留节点

```java
static final class ReservationNode<K,V> extends Node<K,V> {
    ReservationNode() {
        super(RESERVED, null, null, null);
    }

    Node<K,V> find(int h, Object k) {
        return null;
    }
}
```



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、ConcurrentHashMap添加元素操作</h3>

#### 面向用户的 put 方法

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}
```

#### :lollipop::lollipop::lollipop:面向 put 和 putIfAbsent 多用途的添加元素的方法

> 真正实现用于 put 和 putIfAbsent  的添加方法

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    // 不允许key、value为null
    // 将null值判断提前，符合异常处理的规则（这里也是较上一版做了优化）
    if (key == null || value == null) throw new NullPointerException();
    // 计算key的哈希码
    int hash = spread(key.hashCode());
    // binCount 用来记录链表中节点数量，进而判断是否达到转为红黑树的阈值
    int binCount = 0;
    // 遍历table数组
    for (Node<K,V>[] tab = table;;) {
        // 要插入元素所在位置的节点
        Node<K,V> f;
        // n 表示数组长度；i 表示索引；fh 表示要插入元素所在位置的节点的哈希码
        int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            // 如果数组为null或者数组的大小为0，那么进行初始化数组
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // 通过指定key的哈希码找到对应的节点，
            // 在节点为null的情况下，通过CAS自旋方式将这个元素放入其中
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                // 如果通过CAS自旋成功添加元素，就直接跳出循环
                // 否则就进入下一个循环
                break;
        }
        else if ((fh = f.hash) == MOVED)
            // 标识着要迁移数据
            tab = helpTransfer(tab, f);
        // 通过上面过滤的条件，在应该能猜到下面不是关于链表就是关于红黑树
        // 因为哈希槽的位置不为null
        else {
            // 旧值
            V oldVal = null;
            // 只给单个节点加锁
            synchronized (f) {
                // 再次判断节点
                if (tabAt(tab, i) == f) {
                    // 判断节点f的哈希码是否不小于0
                    if (fh >= 0) {
                        // 大于等于0意味着该处有元素，记录元素加1
                        binCount = 1;
                        // 从桶顶（哈希槽）开始遍历链表
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            // 判断key是否相同
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                // 如果相同的话，就准备获取value
                                oldVal = e.val;
                                // onlyIfAbsent为true标识可以覆盖value，false标识不允许覆盖 
                                if (!onlyIfAbsent)
                                    // 如果允许覆盖value，就覆盖value
                                    e.val = value;
                                break;
                            }
                            // 记录下一个节点的上一个节点（目前以为着是当前节点e）
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                // 如果当前节点的下一个节点为null，就把元素添加到链表的尾部
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```















https://github.com/about-cloud/JavaCore
当然后面还会持续更新本文，有兴趣可以关注上面GitHub文章。