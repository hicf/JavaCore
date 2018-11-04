> :dog:本文是基于 `jdk1.7.0_79` 分析

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、非线程安全HashMap</h3>

> 前面[文章](https://github.com/about-cloud/JavaCore)分析了 `HashMap` 源码，但其操作是非现在安全的，比如两个线程并发赋值，其中key相同，而value不相同，就有可能造成值覆盖的情况。再比如一个线程并发删操作、另一个线性并发写操作，也可能造成空转问题。`java.util.concurrent` 包 `ConcurrentHashMap` 是其线程安全的实现。



:family:

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、ConcurrentHashMap的继承关系</h3>

```java
public class ConcurrentHashMap<K, V> extends AbstractMap<K, V>
        implements ConcurrentMap<K, V>, Serializable
```

> 抽象类 `AbstractMap` 实现了一些常用的方法，接口 `ConcurrentMap` 也是继承至 `Map` 接口，它具有并发操作的支持。
>
> 接口 `ConcurrentMap` 定义的未实现方法如下：

```java
// 添加不存在的元素项
V putIfAbsent(K key, V value);
// 删除元素项
boolean remove(Object key, Object value);
// 替换指定key的value，替换成功返回true，否则返回false
boolean replace(K key, V oldValue, V newValue);
// 替换指定key的value
V replace(K key, V value);
```



⛄️⛄️⛄️

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、ConcurrentHashMap数据结构</h3>

> 基于 `jdk1.7` 的 `ConcurrentHashMap`是基于 **分段锁** 机制设计的，此处的锁🔐指的是 `ReentrantLock`。

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、继承关系</h3>