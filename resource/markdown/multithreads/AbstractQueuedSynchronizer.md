<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">并发包的基石 -- 队列同步器 AQS</h3>

#### 本文关注点
* **AbstractOwnableSynchronizer** 抽象的、可拥有的同步器
* `★★★★★`**AbstractQueuedSynchronizer**抽象的、队列(排队)的同步器
* **Sync** 同步
* **FairSync** 公平同步
* **NonfairSync** 非公平同步

---
![AQS](https://upload-images.jianshu.io/upload_images/11476758-15ca6a3e1415a256.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 一、 `AbstractOwnableSynchronizer` 抽象的、可拥有的同步器
![AbstractOwnableSynchronizer的方法与属性](https://upload-images.jianshu.io/upload_images/11476758-f0950e50411f1845.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* **源码分析：**
```java
package java.util.concurrent.locks;

/**
 * 可由线程独占的同步器。
 * 该类为创建锁和相关的同步器提供了基础，可能需要拥有所有权的概念。
 * AbstractOwnableSynchronizer 类本身不管理或使用此信息。
 * 但是，子类和工具可以使用适当维护的值来帮助控制和监视访问并提供诊断。
 *
 * @since 1.6
 * @author Doug Lea
 */
public abstract class AbstractOwnableSynchronizer
    implements java.io.Serializable {

    /** 即使所有的字段都是短暂的（被禁止序列化的），也要使用 序列ID。*/
    private static final long serialVersionUID = 3737899427754241961L;

    /**
     * 供子类使用的空的构造方法。
     */
    protected AbstractOwnableSynchronizer() { }

    /**
     * 独占模式同步下的当前（线程）拥有者。（也就是当前独占锁的线程）
     */
    private transient Thread exclusiveOwnerThread;

    /**
     * 设置 当前拥有独占访问的 线程。
     * 空参数指示没有线程拥有访问权限。
     * 此方法不另外施加任何同步化易失性 volatile 字段访问。
     * @param thread 所有者的线程（拥有当前独占访问权的线程）
     */
    protected final void setExclusiveOwnerThread(Thread thread) {
        exclusiveOwnerThread = thread;
    }

    /**
     * 返回 setExclusiveOwnableThread(Thread thread) 方法最后设置的线程，
     * 如果从未设置，则返回 null。
     * 此方法不另外施加任何同步化易失性 volatile 字段访问。
     * @return 所有者的线程（拥有当前独占访问权的线程）
     */
    protected final Thread getExclusiveOwnerThread() {
        return exclusiveOwnerThread;
    }
}
```
> **小结：**
> **独占、持有锁的线程成员变量：Thread exclusiveOwnerThread**
> **setExclusiveOwnerThread(Thread thread)** 设置独占、持有锁的线程成员变量
> **getExclusiveOwnerThread()** 获得独占、持有锁的线程成员变量


#### 二、 `AbstractQueuedSynchronizer` 抽象、队列同步器 AQS
![AbstractQueueSynchronizer公有属性和公有方法.png](https://upload-images.jianshu.io/upload_images/11476758-10ff5c98c4f752a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![AbstractQueuedSynchronizer 私有方法.png](https://upload-images.jianshu.io/upload_images/11476758-b373d216213b6845.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![AbstractQueuedSynchronizer final域与私有域.png](https://upload-images.jianshu.io/upload_images/11476758-4e2da27a6904ad4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* **AbstractQueuedSynchronizer 简介**
> 提供了一个基于 **FIFO队列**，可以用于构建锁或者其他相关同步装置的基础框架。该 **抽象队列同步器**（以下简称**队列同步器**）利用了一个 **private volatile int state**来表示 **同步状态**，期望它能够成为实现大部分同步需求的基础。使用的方法是继承，子类通过继承同步器并需要实现它的方法来管理其状态，管理的方式就是通过类似 **acquire获取** 和 **release释放** 的方式来 **操纵状态**。然而 **多线程环境中对状态的操纵必须确保原子性**，因此子类对于状态的把握，需要使用这个同步器提供的以下三个方法对状态进行操作：
> * private volatile int **state**;
> * java.util.concurrent.locks.AbstractQueuedSynchronizer.**getState()**
> * java.util.concurrent.locks.AbstractQueuedSynchronizer.**setState(int newState)**
> * java.util.concurrent.locks.AbstractQueuedSynchronizer.**compareAndSetState(int expect, int update)**
* **2.1、源码分析：**
```java
/**
 * 返回同步状态的当前值。
 * 此操作具有易失性 volatile 读取的内存语义。
 * @return 当前 state 的值
 */
protected final int getState() {
    return state;
}

// ------------------- 华丽的分割线 -------------------
/**
 * 设置同步状态 state 的值。
 *  此操作具有易失性 volatile 写的内存语义。
 * @param newState 状态 state 的新值
 */
protected final void setState(int newState) {
    state = newState;
}

// ------------------- 华丽的分割线 -------------------
/**
 * 如果当前状态值等于预期值，则以原子方式将同步状态设置为给定的更新值。
 * 此操作具有易失性 volatile 读取和写入的内存语义。
 *
 * @param expect 期望的值
 * @param update 新值
 * @return 如果成功则为 true 。 错误返回表示实际值不等于预期值的指示。
 */
protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

* **2.2、FIFO 队列** （类似双向链表）
> 同步器的开始提到了其实现依赖于一个 **FIFO 队列**，那么队列中的 **元素节点Node** 就是保存着 **线程引用** 和 **线程状态** 的 **容器**，**每个线程对同步器的访问，都可以看做是队列中的一个节点Node**。Node的主要包含以下成员变量：
```java
static final class Node {
    /** 作为标记表示节点正在 共享模式 中等待 */
    static final Node SHARED = new Node();
    /** 作为标记表示节点正在 独占模式 下等待 */
    static final Node EXCLUSIVE = null;

    int waitStatus;
    volatile Node prev;
    volatile Node next;
    volatile Thread thread;
    volatile Node nextWaiter;

    final boolean isShared() {
            return nextWaiter == SHARED;
    }
}
```
* **2.3、API**

| 方法名称 | 描述 |
| ------ | ------- |
| protected boolean tryAcquire(int arg) | 独占式获取锁，实现该方法需要查询当前状态并判断同步状态是否和预期值相同，然后使用 CAS 操作设置同步状态 |
| protected boolean tryRelease(int arg) |	独占式释放锁，实际也是修改同步变量 |
| protected int tryAcquireShared(int arg)	| 共享式获取锁，返回大于等于 0 的值，表示获取锁成功，反之获取失败 |
| protected boolean tryReleaseShared(int arg) | 共享式释放锁 |
| protected boolean isHeldExclusively()	 | 判断调用该方法的线程是否持有互斥锁 |

* **其他重要方法**
> private Node **addWaiter(Node mode)**

> 参考资料：
> [AbstractQueuedSynchronizer的介绍和原理分析](http://ifeve.com/introduce-abstractqueuedsynchronizer/)

---
##### 三、 `Sync`抽象的静态内部类
![Sync 静态内部类的方法与属](https://upload-images.jianshu.io/upload_images/11476758-9430f3ace4f38a69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 四、 `FairSync` 公平锁（静态内部类）
![FairSync 公平同步器](https://upload-images.jianshu.io/upload_images/11476758-3feff7da83e0a35c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 五、 `NonfairSync` 非公平锁（静态内部类）
![NonfairSync 非公平同步器](https://upload-images.jianshu.io/upload_images/11476758-453881e169960163.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)