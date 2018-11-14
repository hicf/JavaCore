<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">并发辅助工具类（很好的玩的工具类）</h3>



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、Exchanger 交换器（两线程间的通信）</h3>

> **使用场景：**用于 `有且仅有两个线程` 间的 `数据传输`，就就是线程间的 `通信` 。它是 `生产者/消费者` d的 ~~`wait() / notify()`~~ 的最佳替代工具。
> **核心原理**：方法 `exchange()`阻塞特性：此方法被调用后等待其他线程来取数据，如果没有其他线程取得数据，则一直 **阻塞**。
* **示例：交替打印奇偶数：**
```java
public class Print {
    public static void main(String[] args) {
        // 交换器
        Exchanger<Integer> exchanger = new Exchanger<>();
        // 起始打印数值
        Integer startNumber = 1;
        // 终止的数值
        Integer endNumber= 100;

        // 线程1
        Thread t1 = new Thread(new Thread1(exchanger, startNumber, endNumber));
        Thread t2 = new Thread(new Thread2(exchanger, startNumber, endNumber));
        // 设置线程名称
        t1.setName("thread-no-1");
        t2.setName("thread-no-2");
        // 启动线程
        t1.start();
        t2.start();
    }
}

/**
 * 打印奇数的线程
 */
class Thread1 implements Runnable {
    private Exchanger<Integer> exchanger;
    /** 被打印的数 */
    private Integer number;
    private final Integer endNumber;

    public Thread1(Exchanger<Integer> exchanger, Integer startNumber, Integer endNumber) {
        this.exchanger = exchanger;
        this.number = startNumber;
        this.endNumber = endNumber;
    }

    @Override
    public void run() {
        while (number <= endNumber) {
            if (number % 2 == 1) {
                System.out.println("线程：" + Thread.currentThread().getName() + " : " + number);
            }
            try {
                exchanger.exchange(number++);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

/**
 * 打印偶数的线程
 */
class Thread2 implements Runnable {
    private Exchanger<Integer> exchanger;
    /** 被打印的数 */
    private Integer number;
    private final Integer endNumber;

    public Thread2(Exchanger<Integer> exchanger, Integer startNumber, Integer endNumber) {
        this.exchanger = exchanger;
        this.number = startNumber;
        this.endNumber = endNumber;
    }

    @Override
    public void run() {
        while (number <= endNumber) {
            if (number % 2 == 0) {
                System.out.println("线程：" + Thread.currentThread().getName() + " : " + number);
            }
            try {
                exchanger.exchange(number++);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、Semaphore 信号灯</h3>

![信号灯限流](https://upload-images.jianshu.io/upload_images/11476758-b0f7445595d94324.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> **核心原理：** 通过发放设置最大 `许可数`，来限制线程的并发数。
> 默认是 **非公平锁**，效率高。
```java
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

Semaphore semaphore = new Semaphore(5);
try {
    semaphore.acquire(); // 获取许可
    // 逻辑
    
} catch (InterruptedException e) {
    e.printStackTrace();
} finally {
    semaphore.release(); // 释放许可
}
```



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、CountDownLatch 倒计时闩（锁）</h3>

![CountDownLatch](https://upload-images.jianshu.io/upload_images/11476758-d32fe09a960417fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


> **核心原理**：线程以 **组团** 的方式进行任务。
> `count` 作为 `stat 状态`。`await()` 方式将 `阻塞当前线程`，直到 `count 为 0`。
```
CountDownLatch countDownLatch = new CountDownLatch(5);
countDownLatch.countDown(); // count - 1
// 预处理
try {
    countDownLatch.await(); // 阻塞当前线程
    // 大家一起处理的时候，我才处理
} catch (InterruptedException e) {
    e.printStackTrace();
} 
```
* **Sync同步器**
```java
private static final class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = 4982264981922014374L;

    Sync(int count) {
        setState(count);
    }

    int getCount() {
        return getState();
    }

    protected int tryAcquireShared(int acquires) {
        return (getState() == 0) ? 1 : -1;
    }

    protected boolean tryReleaseShared(int releases) {
        // 递减 count; 转换为零时发出信号
        for (;;) {
            int c = getState();
            if (c == 0)
                return false;
            int nextc = c-1;
            if (compareAndSetState(c, nextc))
                return nextc == 0;
        }
    }
}
```



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">四、CyclicBarrier 循环栅栏（循环锁）</h3>

> **核心原理：** 基于 `ReentrantLock` 和 `Condition`。
> `CyclicBarrier` 不仅具有 `CountDownLatch` 的功能，还有实现屏障等待的功能，也就是阶段性同步。

* **CyclicBarrier与CountDownLatch比较**
  * 1）CountDownLatch:一个线程(或者多个)，等待另外N个线程完成某个事情之后才能执行；CyclicBarrier:N个线程相互等待，任何一个线程完成之前，所有的线程都必须等待。
  * 2）CountDownLatch:一次性的；CyclicBarrier:可以重复使用。
  * 3）CountDownLatch基于AQS；CyclicBarrier基于锁和Condition。本质上都是依赖于volatile和CAS实现的。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">五、Phaser 移相器</h3>





