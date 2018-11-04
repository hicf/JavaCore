:oncoming_police_car::oncoming_police_car::oncoming_police_car:

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、非线程安全的HashMap</h3>

> :clapper:前面的[章节](https://github.com/about-cloud/JavaCore/)已经从源码分析了 `HashMap` 的实现，这里简单回顾一下，如想详细了解，[请参考这里](https://github.com/about-cloud/JavaCore)。
>
> * `jdk1.7` 的 `HashMap` 的底层实现是 **数组** + **单向链表**，支持key为 `null`，value 为`null`。
> * 它的初始容量是 `16`，负载因子默认值 `0.75`，当实际存放元素数量大于等于 **阈值**，（而且新元素被放入的哈希槽不为空），那么就会触发扩容，每次扩容时的容量都是翻倍增长（一定是 `2`的次幂）,所以在实例存储元素项较多的情况下，一定要指定初始容量，避免每次扩容带来性能上的影响。
> * 其属性、方法、代码块没有被 `synchronize`修饰，也没有使用其他同步机制，在多线程环境下是 **非线程安全**的。
> * **负载因子** 取值问题也是一个重要原因，取值越大**哈希冲突** 的概率就越高，取值越小空间浪费度就越高。而 `0.75` 是一个比较折中的选择。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、线程安全的Hashtable</h3>

> 在 **上古时期**的容器中，`Hashtable`就是其中一个线程安全的容器，可追溯到` jdk1.0`。

```java
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable
```

> `Hashtable` 扩展至 `Dictionary` 这个抽象类。



:suspension_railway::suspension_railway::suspension_railway:

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、线程安全的ConcurrentHashMap</h3>