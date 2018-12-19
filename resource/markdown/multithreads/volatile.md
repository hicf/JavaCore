<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">Java内存模型之：volatile</h3>



#### 概念
> 把对 **volatile变量**的单个读/写，看成是使用 **同一个监视器锁** 对这些单个读/写操作做了 **同步**。
> 原理：插入内存屏蔽指令，禁止一定条件下的重排序。
* **volatile** 是轻量级的同步机制
  


* 举例说明：
```
public class Assignment {
    int value = 1;
    /**
     * 加法
     */
    public void assign1() {
        value = 1; // 单操作
    }
    public void assign2() {
        value = 2; // 单操作
    }
}
```
* volatile禁止指令重排序也有一些规则，简单列举一下**（重点是存在多操作）**：
> * 1. 当第二个操作是voaltile写时，无论第一个操作是什么，都不能进行重排序
> * 2.当地一个操作是volatile读时，不管第二个操作是什么，都不能进行重排序
> * 3.当第一个操作是volatile写时，第二个操作是volatile读时，不能进行重排序


> 参考资料：
> [深入理解Java内存模型（四）——volatile](http://www.infoq.com/cn/articles/java-memory-model-4)