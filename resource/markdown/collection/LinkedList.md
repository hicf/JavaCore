### LinkedList 源码分析

> 源码分析基于 `jdk-11.0.1`

### 本文重点

#### 添加元素

#### 删除元素

---

### 一、扩展关系

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

`LinkedList` 继承至 `AbstractSequentialList`表现其具有连续性，同时实现了线性的 `List` 和双端的 `Deque`，**LinkedList** 底层实现就是双向链表。



### 二、源码分析

#### 重点属性

```java
// 记录元素数量
transient int size = 0;
// 记录链表更改的次数
protected transient int modCount = 0;
// 指向头节点
transient Node<E> first;
// 指向尾节点
transient Node<E> last;

// 链表中的每个节点
private static class Node<E> {
    // 节点中存放的元素
    E item;
    // 前驱节点
    Node<E> next;
    // 后继节点
    Node<E> prev;

    // 节点内唯一的方法(构造方法)来注入元素、前驱节点、后继节点
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```



#### 2.1、添加元素

#### 添加元素到链表尾部

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}

// 默认将元素添加到双向链表的尾部
void linkLast(E e) {
    // 将原来的最后一个节点存储起来
    final Node<E> l = last;
    // 实例化一个新的节点，也就是被链入的节点，
    // 并通过Node唯一的构造方法注入前驱节点和元素
    // (因为它被链入尾部，所以后面是没有节点的)
    final Node<E> newNode = new Node<>(l, e, null);
    // 将新构建的节点链入尾部
    last = newNode;
    // 如果链表为空，那么它也是头节点
    if (l == null)
        first = newNode;
    else
        l.next = newNode; // 否则，刚才那个节点的后继节点就是这个新节点
    // 记录元素个数加 1
    size++;
    // 更改次数加 1
    modCount++;
}
```

添加元素到头部，操作于此类似。



#### 重点★★★★★ 添加元素到指定index位置

```java
public void add(int index, E element) {
    // 和 ArrayList 相似，只要遇到 index ，必做下标越界检查
    // 这也符合异常处理规范：如有异常提前抛出
    checkPositionIndex(index);
    // 如果正好是链表的尾部之后，则链入尾部
    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index)); // 否则链入 index 位置元素的前面
}
// 在正式链表前，还需要计算index处的节点(或者说是获取index处的节点)
// 前面说过，LinkedList 实现至 List接口，它也有List线性的特性，可以进行迭代
// 这个方法有个牛逼的地方，请看下方
Node<E> node(int index) {
    // assert isElementIndex(index);
    // 判断 index “距离”哪头近，就从哪头开始遍历，这样可以节省时间
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}

/**
 * @param e 被链入的元素
 * @param succ index 位置的元素(下文成其为succ节点 -- 也就是新节点的后继节点)
 */
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    // 获取 succ节点的前驱节点引用
    final Node<E> pred = succ.prev;
    // 构造新的、即将被链入的节点
    final Node<E> newNode = new Node<>(pred, e, succ);
    // 将新节点设置为succ节点的前驱节点
    succ.prev = newNode;
    
    // 以下操作就是不要忘记头节点、尾节点、size、modCount
    // 如果succ节点没有前驱节点，那么新链入的节点就是头节点
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode; // 否则，将新节点设置成之前succ的前驱节点的后继节点
    size++;
    modCount++;
}
```

#### 图解双向链表的添加模式：

![链表添加元素前](http://pgq1yfr0p.bkt.clouddn.com/image/java/collection/LinkedList1.png)

**第一步：**通过 `Node<E> node(int index)` 获取 `succ 节点`（假设index为3）

![使用node(index)方法获取succ节点](http://pgq1yfr0p.bkt.clouddn.com/image/java/collection/LinkedList2.png)

**第二步：**链表添加元素

```java
// succ 前驱节点是2节点，pred节点实际指向2节点指向的地址
① final Node<E> pred = succ.prev;
// 构造新的节点，同时又设置前驱节点为 pred(也就是指向2节点)、后继节点为succ
② final Node<E> newNode = new Node<>(pred, e, succ);
// 设置succ的前驱节点为新节点
③ succ.prev = newNode;
④ if (pred == null) // 这里不介绍 pred 为空的情况
      first = newNode;
  else
      pred.next = newNode; // 设置 pred节点（也就是2节点）的后继节点为新节点
```

![链入元素](http://pgq1yfr0p.bkt.clouddn.com/image/java/collection/LinkedList4.png)

#### 2.2 删除元素

```java
public boolean remove(Object o) {
    // 分两种情况：被删除的元素为nul与不为空
    if (o == null) {
        // for 循环判断
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}

// 解链方法，删除节点方法
E unlink(Node<E> x) {
    // assert x != null;
    // 被删除节点的元素
    final E element = x.item;
    // 被删除节点的后继节点
    final Node<E> next = x.next;
    // 被删除节点的前驱节点
    final Node<E> prev = x.prev;

    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null; // 空引用，利于GC回收“无用”的对象
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null; // 空引用，利于GC回收“无用”的对象
    }

    x.item = null; // 空引用，利于GC回收“无用”的对象
    size--;
    modCount++;
    return element;
}
```

#### 图解删除LinkedList中的节点

> 假设删除是3节点（这里不介绍头结点和尾节点为空的情况）

首先获取 `被删除节点x（既节点3）` 的 `后继节点next（既节点4）` 和 `前驱节点prev（既节点2）`;

然后将prev节点（2节点）的next指向next节点（4节点）；

然后将next节点（4节点）的prev指向prev节点（2节点）；

此时就完成，删除操作要比添加操作更为简单。

![图解删除LinkedList中的节点](http://pgq1yfr0p.bkt.clouddn.com/image/java/collection/LinkedList5.png)