> 本文是基于 `jdk1.7.0_71` 分析

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、Map接口</h3>

> **Map接口**是整个 **Key-Value** 存储容器的核心。它 **抽**(qú)(xíng)**象** 定义了 **Key-Value** 结构的容器框架。**Map**不会像 **数组** 中元素那样可以单独存放，到像似 `LinkedList` 中的节点自定义，那么特殊的节点一定需要特定的来描述。在 `jdk1.7` 及其之前的版本，Map的作者将其节点描述为 **Entry** ，Entry不是 ~~入口~~、~~大门~~，它在这里称为 **项目** 或 **条目** ，下文简称为 **项**。**Map** 中的每个节点称为 `一项`。

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、Map.Entry</h3>

> **Entry<K, V>** 是Map内部的接口，其中定义了一些方法：

```java
// 返回此项对应的key
K getKey();
// 返回此项对应的value
V getValue();
// 设置此项中value值
V setValue(V value);
// 将指定的对象与此项进行比较
boolean equals(Object o);
// 返回此项的哈希码值
int hashCode()
```

> Entry 项 的定义很简单，主要是记住它是 Key-Value 形式，并有一些get/set方法。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、Map中的方法</h3>

> **方法** 意味着 行为。下面Map的方法意味着整个Map框架具有最基本的什么样的 **功能**。明白这些方法，就掌握了Map框架核心操作功能。

```java
// 返回 map 中键值对的数量
int size();
// 返回 true 表示map中不包含键值对
boolean isEmpty();
// 判断比较map中是否含有指定对象相等的key，如果含有返回true
boolean containsKey(Object key);
// 判断比较map中是否含有指定对象相等的value，如果含有返回true
boolean containsValue(Object value);
// 通过key国外value
V get(Object key);
// 放入 key-value键值对，并返回value
V put(K key, V value);
// 通过指定key的一项，并返回此项中的value
V remove(Object key);
// 放入另一个map
void putAll(Map<? extends K, ? extends V> m);
// 清空map
void clear();
// 返回map中key的Set集合
Set<K> keySet();
// 将map中的value以Collection的形式返回
Collection<V> values();
// 返回map中多个项的Set集合(此方法适合既获取key又获取value的场景)(请记住这个方法，非常实用)
Set<Map.Entry<K, V>> entrySet();
// 将指定的对象与map项进行比较
 boolean equals(Object o);
// 返回map的哈希码
int hashCode();
```



> :herb:合抱之木,生于毫末;:mount_fuji:九层之台,起于累土;:camel:千里之行,始于足下。
>
> :dart:再次提醒，请熟记这次基础方法。