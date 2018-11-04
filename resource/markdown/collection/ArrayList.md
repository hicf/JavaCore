<h4 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#2C3E50;">本文关注点：</h4>

#### **ArrayList**
#### **Vector** 和 **Stack**
#### `★★★★★`本文重点：ArrayList自动扩容机制（**1.5倍或1.5倍-1** 扩容）

***
<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、ArrayList</h3>

> ArrayList是动态数组，其动态性体现在能够`动态扩容、缩容`。（其原理是构建一个新Object数组，将原数组复制进去。借助 `Arrays.copyOf(T[] original, int newLength)）` 。

> ![ArrayList继承关系](https://upload-images.jianshu.io/upload_images/11476758-6749a7cf5848ce66.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####  重要的属性:ledger:
```java
// 默认初始容量为 10（当ArrayList的容量低于9时才会用到）
private static final int DEFAULT_CAPACITY = 10;
// 存放数据的数组，数组的每个位置并不一定都填充数据，用transient修饰避免序列化、避免浪费资源
transient Object[] elementData;
// 记录实际存放的元素个数
private int size;
// 记录着ArrayList的修改次数,也就每次add或remove，它的值都会加1
protected transient int modCount = 0;
```

#### 1.1、一般添加元素的方法 public boolean `add(E e)`
```java
public boolean add(E e) {
    // modCount++，并且校验容量，不够用就扩容
    ensureCapacityInternal(size + 1);  // 增加 modCount
    // 将新元素添加到新数组size位置，并将size加1（千万不要理解成将新元素添加到size+1的位置）
    elementData[size++] = e;
    return true;
}

// 确保容量够用
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // 下标溢出表示容量不够用
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

```
#### `本文重点方法★★★★★` ArrayList 扩容方法 grow(int minCapacity)
```java
// 扩容方法
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    // ArrayList 的扩容是 1.5 倍 或 1.5 倍 - 1 
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // 该拷贝为深拷贝，将原来的数组复制到新的数组并返回新的数组
    elementData = java.util.Arrays.copyOf(elementData, newCapacity);
}
```

:bomb::bomb::bomb:虽然上面的代码不多，但是添加元素遇到经常扩容的现象要格外留心，因为频繁的扩容势必带来频繁的数组拷贝，这会大大牺牲性能，所以有必要在 `ArrayList` 初始化时指定容量。初始化的容量也不要太大于实际存储容量，不然会造成空间浪费，所以都要权衡利弊。

#### 1.2、在指定位子添加方法 public void `add(int index, E element)`

```java
public void add(int index, E element) {
    // 只要有index，必定会检查range
    rangeCheckForAdd(index);
    // 确保容量够用（否则就扩容）
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 这里是在指定位置index添加元素的关键：从index处，将数据后移 1 位，空出index处给新元素用
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    // 上一步已将index处的位置空出来，现在将新元素添加到index处
    elementData[index] = element;
    // 实际存储的元素数加1
    size++;
}
```

#### 2.1、删除元素 public boolean `remove(Object o)`
```java
public boolean remove(Object o) {
    // 两种情况，被删除的元素是否为 null
    // for 循环遍历、判断、删除（请注意：千万不要使用 foreach 进行删除！！！）
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    // 如果删除了，返回为true，否则返回false。
    return false;
}

/*
 * Private remove method that skips bounds checking and does not
 * return the value removed.
 * 跳过边界检查的私有移除方法，并且不返回删除的值
 */
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        // 重点在这里，数据向前移动
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    // 清除 size-1 位置上的元素（设置空引用），让GC去收集该元素，同时 size 减 1
    elementData[--size] = null; 
}
```

#### 3、重新设置指定index位置的元素值 public E `set(int index, E element)`
```java
public E set(int index, E element) {
    // 有index，必检查
    rangeCheck(index);
    // 返回旧的（被重置的）元素
    E oldValue = elementData(index);
    // 直接把 index位置的元素覆盖掉行了
    elementData[index] = element;
    return oldValue;
}
```

#### 4、获取指定index位置的元素 public E `get(int index)` 
```java
public E get(int index) {
   // 还是那句话，凡是遇到index索引，必检查
    rangeCheck(index);
    // 直接返回index的元素，数组查询还不快吗。
    return elementData(index);
}

E elementData(int index) {
    // 因为 ArrayList 本质上是个数组，直接通过index获取元素
    return (E) elementData[index];
}
```

#### 5、判断元素是否存储 public boolean `contains(Object o)` 判断是否存在某个元素
```java
public boolean contains(Object o) {
    // 简单粗暴，直接遍历查看index是否大于0
    return indexOf(o) >= 0;
}

// 查找元素是否存在，如果存在返回下标，否则返回 -1
public int indexOf(Object o) {
    // 遍历的时候分两种情况，是否为 null
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            // 用 equals 判断值是否相等
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

***
<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、ArrayList的亲兄弟 Vector</h3>

> Vector 的大部分方法是被 `synchronized` 修饰的。
线程安全的ArrayList，几乎所有的方法都加 `synchronized` 修饰；

***
<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、Stack</h3>

> Stack 继承至 Vector，是一个`后进先出LIFO`的队列。
它实现了一个标准的 `后进先出LIFO` 的栈。