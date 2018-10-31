> 本文是基于 `jdk1.7.0_79` 分析

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、继承关系</h3>

![HashMap1.7vExtend](http://pgq1yfr0p.bkt.clouddn.com/image/java/collectionHashMap1.7vExtend.png)

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、数据结构</h3>

>底层结构：数组+线性链表

#### 重要属性

```java
/** 默认初始容量 16 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // 既 16
/** 最大容量 10.7亿+（这里也用到效率高的位运算） */
static final int MAXIMUM_CAPACITY = 1 << 30;
/** 默认负载因子 0.75 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;
/** 默认实际存储元素数量的阈值 8 */
static final int TREEIFY_THRESHOLD = 8;
/**  */

```



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、添加元素</h3>



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">四、删除元素</h3>

