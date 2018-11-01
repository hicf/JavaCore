> 本文是基于 `jdk1.7.0_79` 分析

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、继承关系</h3>

![HashMap1.7vExtend](http://pgq1yfr0p.bkt.clouddn.com/image/java/collectionHashMap1.7vExtend.png)

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、数据结构</h3>

>底层结构：数组+线性链表

#### 重要属性（请记住这些常量和变量的含义，下面源码分析会用到）

```java
/** 默认初始容量 16 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // 既 16
/** 最大容量 10.7亿+（这里也用到效率高的位运算） */
static final int MAXIMUM_CAPACITY = 1 << 30;
/** 默认负载因子 0.75 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;
/** 当 table 未填充的时候，共享这个空table */
static final Entry<?,?>[] EMPTY_TABLE = {};
/** 默认实际存储元素数量的阈值 8 */
static final int TREEIFY_THRESHOLD = 8;
/** 该 table 根据需要而调整大小。长度必须始终是2的次幂。 */
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
/** map 中 存储 key-value 映射的数量 */
transient int size;
/** 下次调整大小时的阈值(容量 * 负载因子) */
int threshold;
/** 哈希表的负载因子 */
final float loadFactor;
/** 此 HashMap 修改的次数 */
transient int modCount;
/** 默认容量的最大值：Integer.MAX_VALUE */
static final int ALTERNATIVE_HASHING_THRESHOLD_DEFAULT = Integer.MAX_VALUE;
```

`HashMap` 重点元素节点

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

    public final K getKey() {
        return key;
    }

    public final V getValue() {
        return value;
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry e = (Map.Entry)o;
        Object k1 = getKey();
        Object k2 = e.getKey();
        if (k1 == k2 || (k1 != null && k1.equals(k2))) {
            Object v1 = getValue();
            Object v2 = e.getValue();
            if (v1 == v2 || (v1 != null && v1.equals(v2)))
                return true;
        }
        return false;
    }

    public final int hashCode() {
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }

    public final String toString() {
        return getKey() + "=" + getValue();
    }

    /**
     * This method is invoked whenever the value in an entry is
     * overwritten by an invocation of put(k,v) for a key k that's already
     * in the HashMap.
     */
    void recordAccess(HashMap<K,V> m) {
    }

    /**
     * This method is invoked whenever the entry is
     * removed from the table.
     */
    void recordRemoval(HashMap<K,V> m) {
    }
}
```



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、添加元素</h3>



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">四、删除元素</h3>

