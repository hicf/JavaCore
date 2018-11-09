> 本文是基于 `jdk1.8.0_141` 分析
>
> 前面已经基于 `jdk1.7` 源码分析了 `HashMap`，文章地址：https://github.com/about-cloud/JavaCore
>
> `jdk1.8` 在之前的基础上做了很多优化、更新(比如使用了函数式编程-拉姆达λ表达式、引入了红黑树)。

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、继承关系</h3>

> 扩容关系上没有变化，但被扩展的父类(接口)也增加了一些方法，见下面：

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
```

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、Map接口</h3>

#### `jdk1.8` 的 `Map<K, V>` 接口相较于 `jdk1.7` 增加了如下方法：

```java
/**
 * 该静态方法返回按 key 进行比较、自然排序的比较器。
 * 如果 key 为 null 会抛出 NullPointerException
 *
 * @since 1.8
 */
public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K,V>> comparingByKey() {
    return (Comparator<Map.Entry<K, V>> & Serializable)
            (c1, c2) -> c1.getKey().compareTo(c2.getKey());
}

/**
 * 该静态方法返回按 value 进行比较、自然排序的比较器。
 * 如果 value 为 null 会抛出NullPointerException
 *
 * @since 1.8
 */
public static <K, V extends Comparable<? super V>> Comparator<Map.Entry<K,V>> comparingByValue() {
    return (Comparator<Map.Entry<K, V>> & Serializable)
            (c1, c2) -> c1.getValue().compareTo(c2.getValue());
}
```

:recycle: 上述方法还表明：从 `jdk1.8` 开始，接口中也支持定义 **静态方法**，但静态方法必须有 **方法体**（亲测有效）。

如果想在接口中定义一个有**方法体** 的 **成员方法** 怎么办？从`jdk1.8` 开始也加入了支持，只需要用 `default` 关键字修饰成员方法就可以了，这样做的好处，不用在其子类进行逐个实现（感觉越来越像抽象类了）。如下（还是在Map接口中）：

```java
/** 添加不存在指定 key 的键值对 */ 
default V putIfAbsent(K key, V value) {
    V v = get(key);
    if (v == null) {
        v = put(key, value);
    }

    return v;
}

/** 
 * 删除节点的方法
 * 非同步方法、非原子操作。子类想要此方法具有同步性，那必须要重写此方法。（下同）
 */
default boolean remove(Object key, Object value) {
    Object curValue = get(key);
    if (!Objects.equals(curValue, value) ||
            (curValue == null && !containsKey(key))) {
        return false;
    }
    remove(key);
    return true;
}

/**
 * 重置已存在的指定 key，value 的 value
 */
default boolean replace(K key, V oldValue, V newValue) {
    Object curValue = get(key);
    if (!Objects.equals(curValue, oldValue) ||
            (curValue == null && !containsKey(key))) {
        return false;
    }
    put(key, newValue);
    return true;
}

/**
 * 重置已存在的指定 key 的 value
 */
default V replace(K key, V value) {
    V curValue;
    if (((curValue = get(key)) != null) || containsKey(key)) {
        curValue = put(key, value);
    }
    return curValue;
}
/** 其他略 */
```

:pencil: **AbstractMap<K,V>** 接口在 `jdk1.8` 略有改动，没太大差别。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、数据结构</h3>

> `jdk1.8` 的`HashMap`  优化主要体现在数据结构上，为什么要这样优化呢？[参考前篇文章中的:palm_tree:树形数据结构](https://github.com/about-cloud/JavaCore)
>
> 数据结构：**数组** + **链表** + **红黑树**（:curly_loop:我更喜欢这样描述：数组 + 链表、数组 + 红黑树）

![jdk1.8HashMap]()

#### 重要属性

```java
/** 默认初始容量 16 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // 既 16
/** 最大容量 10.7亿+（这里也用到效率高的位运算） */
static final int MAXIMUM_CAPACITY = 1 << 30;
/** 默认负载因子 0.75 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;
/** （从链表上的元素个数来讲）链表转红黑树的阈值 */
static final int TREEIFY_THRESHOLD = 8;
/** （从红黑树上的元素个数来讲）红黑树转链表的阈值 */
static final int UNTREEIFY_THRESHOLD = 6;
/** 链表转红黑树的另一个阈值(条件)：Map的容量至少为64. */
static final int MIN_TREEIFY_CAPACITY = 64;

/**
 * （这个表又称为基本表、散列表、哈希表）该 table 根据需要而调整大小。
 * 长度必须始终是2的次幂。
 * 这里一个优化点：只有在使用时才初始化。
 */
transient Node<K,V>[] table;
/** 节点中的元素节点的Set集 */
transient Set<Map.Entry<K,V>> entrySet;
/** map 中 存储 key-value 映射的数量 */
transient int size;
/** 此 HashMap 修改的次数 */
transient int modCount;
/** 下次调整大小时的阈值(容量 * 负载因子) */
int threshold;
/** 哈希表的负载因子 */
final float loadFactor;
```

#### 🌞 大多数用于存储元素的 Node<K, V> 节点（链表必用的节点）

>  `HashMap` 重点元素 **项Entry<K, V>** 在 `jdk1.8` 已改为 **节点Node<K, V>**，但它还是实现于 `Map.Entry` 接口，下面就来分析一下 `jdk1.8` `HashMap`实现`Map.Entry` 的 **Node<K, V>**：
>
> :underage:请注意：这个**Node<K, V>** 是**大多数** 用于存储的元素节点，并不是全部，而**红黑树** 是用下面的 **TreeNode<K, V>** 节点作为元素存储节点。因为 **链表** 中的每个节点只有一个后继节点，而 **TreeNode<K, V>** 作为二叉树中的节点，最多可有两个后继节点（既左、右子节点）。

可参考[往期的文章](https://github.com/about-cloud/JavaCore)：https://github.com/about-cloud/JavaCore

```java
static class Node<K,V> implements Map.Entry<K,V> {
    // final 修饰的 哈希码、key，防止被重复赋值
    final int hash;
    final K key;
    // 可被重复设置值的value
    V value;
    // 当前节点的下一个节点(用于链表)
    Node<K,V> next;
	/**
     * 构造方法用于注入 node节点 的属性值(或引用)
     * 参数从左至右依次是：key的哈希码，key，value，指向的下一个节点next
     */
    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
	// getter & toString 方法
    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }
	// 返回节点的哈希码
    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }
	// 设置节点的新值value
    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }
	// 比较节点的方法
    public final boolean equals(Object o) {
        if (o == this)
            // 引用地址或字面量相同即为同一个元素
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

#### 🎉专用于红黑树的 TreeNode<K, V> 节点

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
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
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }

    /**
     * 用户搜索并返回根节点
     */
    final TreeNode<K,V> root() {
        // 循环网上搜索父节点
        for (TreeNode<K,V> r = this, p;;) {
            if ((p = r.parent) == null)
                return r;
            r = p;
        }
    }
}
```



// compare方法 TODO

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">四、添加元素节点</h3>

#### 面向用户的put方法

```java
/**
 * 将指定key、value添加到容器中，
 * 如果映射之前就包含了此处的key，则直接替换此处的value
 */
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

#### :triangular_flag_on_post:实现 `Map.put` 及其相关的添加方法

```java
/**
 * @param hash key的哈希码
 * @param key 指的key
 * @param value 添加的value
 * @param onlyIfAbsent 当其为true时, 如果指的key已存在，不会覆盖已存在的 value
 * @param evict 当其为false, 表处于创建模式（用于LinkedHashMap中的尾部操作，这里没有实际意义。）
 * @return 之前的value，如果之前没有此元素，则为null
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    // 数组(基本表、散列表)
    Node<K,V>[] tab;
    // 指定的key存在的话，那么此处的节点p会暂时地指向该key落在哈希槽（桶顶）的元素
    Node<K,V> p;
    // n 用于记录数组的长度
    int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        // 如果基本表为null或者表的长度为0，那么就扩容，并获取扩容后的数组长度
        n = (tab = resize()).length;
    // 这里是集成了多操作，其中一个是获取指定key落在哈希桶桶顶的元素
    if ((p = tab[i = (n - 1) & hash]) == null)
        // 如果key落在的哈希槽的位置为null，则在该处创建一个新的节点（用于存放指定的键值对）
        tab[i] = newNode(hash, key, value, null);
    else {
        // 到这里表明，指定的key所落在的位置有元素，那么开始遍历链表或红黑树
        // 当前表示记录与指定key相同的节点，后面链表中表示，当前节点p的下一个节点e
        Node<K,V> e;
    	// 桶顶元素的key
        K k;
        // 判断桶顶的元素与指定（将要添加的）这个元素的key是否相同
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            // 如果key相同，就记录此节点
            e = p;
        // 如果key不相同，判断是否是红黑树中的节点
        else if (p instanceof TreeNode)
            // 如果是红黑树中的节点话，就按照红黑树添加节点的方式添加节点
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 既不为空节点，又不是红黑树中的节点，
            // 那么就是普通Node<K, V>节点（这里可以认为是链表中的节点了）
            // 遍历链表
            // binCount 用来记录链表中节点数量，进而判断是否达到转为红黑树的阈值
            for (int binCount = 0; ; ++binCount) {
                // 获得链表中当前节点p的下一个节点e
                // 并判断下一个节点e是否为null
                if ((e = p.next) == null) {
               		// 如果next为null，意味着当前节点是链表的尾节点
                    // 那么就在尾节点后面链入新节点（区别于jdk1.7中添加方式）
                    p.next = newNode(hash, key, value, null);
                    // 判断链表中的节点数binCount是否达到转为红黑树的阈值
                    if (binCount >= TREEIFY_THRESHOLD - 1)
                        // 如果达到就转为红黑树
                        treeifyBin(tab, hash);
                    break;
                }
                // 如果当前节点的下一个节点不为null，
                // 就判断当前节点的哈希码、key、key的哈希码与指定键值对的是否相同，
                // 如果相同，就直接退出链表的遍历
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                // 如果不相同，就遍历下一个节点
                p = e;
            }
        }
        // 如果e不为null，表明指定的key存在对应的现有的键值对
        // 那么就替换value
        if (e != null) {
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            // 空方法，用于扩展
            afterNodeAccess(e);
            // 返回被替换的oldValue
            return oldValue;
        }
    }
    // 如果添加成功，modCount加1
    ++modCount;
    // 添加此元素后，size加1，并判断添加此元素后，实际存储元素的数量是否超过阈值
    // 如果超过阈值，则扩容
    if (++size > threshold)
        resize();
    // 空方法
    afterNodeInsertion(evict);
    return null;
}
```

#### :black_circle::red_circle::palm_tree:将元素添加至红黑树

```java
/**
 * @param map 所在的map
 * @param tab 所在的数组表
 * @param h 节点的哈希码（非key的哈希码）
 * @param k 指定的key
 * @param v 指定的value
 * @return 指定key所匹配到的元素节点
 */
final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab,
                               int h, K k, V v) {
    // key的类对象
    Class<?> kc = null;
    // 标记是否查找过此树
    boolean searched = false;
    // 根节点（父节点不为null，就查找并返回父节点，否则自己就是根节点）
    TreeNode<K,V> root = (parent != null) ? root() : this;
    // 从根节点开始遍历红黑树
    for (TreeNode<K,V> p = root;;) {
        // dir：搜索的方向，左或右：-1表示左，1表示右，
        // ph：当前节点的哈希码，
        // pk：当前节点的key
        int dir, ph; K pk;
        // 如果当前节点的哈希码大于被添加元素的哈希码
        // 那么向左搜索
        if ((ph = p.hash) > h)
            dir = -1;
        // 如果当前节点的哈希码小于于被添加元素的哈希码
        // 那么向右搜索
        else if (ph < h)
            dir = 1;
        // 如果当前节点的key与指定被添加元素的key相同
        // 那么表示已存在指的key的节点，并返回此节点
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            return p;
        else if ((kc == null &&
                // 判断 k 的类是否实现了比较器
                (kc = comparableClassFor(k)) == null) ||
                 // 这里实际是 (pk == null || pk.getClass() != kc ? 0 
                 // : ((Comparable)k).compareTo(pk))
                 // 下面是解读这个三目运算：
                 // pk == null 表示判断当前节点是否为null
                 // pk.getClass() != kc 表示当前节点对象的类和key的对象的类是否不同
                 // ((Comparable)k).compareTo(pk)表示将指定的key与当前节点的key比较
                (dir = compareComparables(kc, k, pk)) == 0) {
            if (!searched) {
                // 判断是否搜索过
                // 如果没有的话，开始搜索
                // q表示需要返回的节点，ch表示子节点
                TreeNode<K,V> q, ch;
                // 标识已经搜索过
                searched = true;
                // find 进行查找，内部也会递归搜索
                if (((ch = p.left) != null &&
                     (q = ch.find(h, k, kc)) != null) ||
                    ((ch = p.right) != null &&
                     (q = ch.find(h, k, kc)) != null))
                    return q;
            }
            // 比较当前节点的key和指定key的大小
            dir = tieBreakOrder(k, pk);
        }
		// 当前节点
        TreeNode<K,V> xp = p;
        // 判断向左还是右搜索，判断左子节点(或右子节点)是否为null
        if ((p = (dir <= 0) ? p.left : p.right) == null) {
            // 如果左子节点(或右子节点)为null，
            // 就将元素插入进来
            Node<K,V> xpn = xp.next;
            TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);
            // 判断并设置当前新节点该为当前节点的左子节点还是右子节点
            if (dir <= 0)
                xp.left = x;
            else
                xp.right = x;
            // 当前节点
            xp.next = x;
            x.parent = x.prev = xp;
            if (xpn != null)
                ((TreeNode<K,V>)xpn).prev = x;
            // 平衡二叉树
            moveRootToFront(tab, balanceInsertion(root, x));
            return null;
        }
    }
}
```



#### :heavy_plus_sign::heavy_plus_sign::heavy_plus_sign:扩容方法 resize

> 有了上面添加元素的方法，扩容方法不难。思路如下：2倍于原容量扩容，并将原容器的元素一个个放入新容器（扩容后的容器）中，那么就用到上面的方法了。（和其他容器一样要注意扩容带来性能的影响，可参看往期文章）

```java
final Node<K,V>[] resize() {
    // 当前数组（基本表、哈希表。散列表）
    Node<K,V>[] oldTab = table;
    // 当前散列表的容量（如果当前散列表为null，那么容量为0，否则为当前散列表的长度）
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    // 当前散列表的阈值
    int oldThr = threshold;
    // 新散列表的容量，新散列表的阈值
    int newCap, newThr = 0;
    // 判断当前散列表的容量
    if (oldCap > 0) {
        // 如果散列表的容量大于0，并且大于等于散列表所允许的最大容量
        if (oldCap >= MAXIMUM_CAPACITY) {
            // 那么将阈值职位 Integer 最大值21.47亿+
            threshold = Integer.MAX_VALUE;
            // 并返回当前散列表
            return oldTab;
        }
        // 如果散列表的容量大于0，并且小于散列表所允许的最大容量
        // 同时获取2倍扩容后的容量，并判断新容量是否大于
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
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



:dart::dart::dart: 上面已经看出每次扩容时都伴随着所有元素项的重新 **选址** 存放，这不仅大大牺牲性能，而且耗时。所以在使用 `HashMap` 一定要评估存放元素项的数量，指定map的大小。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">四、删除元素项</h3>

> 删除元素项的的思路基本和添加操作相似，只不过一个是添加，一个是删除。先根据 `key` 计算 `哈希槽`(桶的位置)，然后判断是链表还是红黑树，根据各自的规则查找。

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">其他问题</h3>

#### 为什么 HashMap 的容量必须是2的次幂呢:question:

> `key`的哈希码对 基本表 做 **逻辑与** 运算 **h&(length-1)**，来确定此元素项的数组上的位置。 原因就出在 **二进制** 的 **与&** 操作上了。
>
>  **与&** 运算规则：`0&0=0`;  `0&1=0`;   `1&0=0`;    `1&1=1`;

| （指数形式）length | （十进制）length | （十进制）length - 1 |（二进制）length - 1|
| ---------------- | ------------------ | -------------------- |--|
| $$1^2$$| 2  | 1 | 1 |
| $$2^2$$| 4  | 3 | 11 |
| $$3^2$$| 8  | 7 | 111 |
| $$4^2$$ | 16  | 15 | 1111 |
| $$...^2$$| ...  | ... | ... |

> 这种情况下，`length - 1` 二进制的 **最右位** 永远是 `1`。这样 `0 & 1 = 0`，`1 & 1 = 1` 的结果既有 `0` 又有`1`。对于 **哈希槽** 的二进制最右位为 `1` （十进制的奇数）的位置就有可能被填充，而不至于浪费存储空间。
>
> 如果 `HashMap` 的容量不是 `2` 的次幂，比如 容量length为 `19` ，`length - 1` 的二进制为 `10010`，任何常数与之作 **与&** 运算，二进制最右位都是 `0`，那么只能存放 （十进制）尾数为 `偶数` 的数组位置，这样会大大浪费空间。