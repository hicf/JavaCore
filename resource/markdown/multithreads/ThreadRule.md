

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、重排序</h3>

![排序](https://i.loli.net/2018/12/19/5c19f0bccee8e.jpeg)

前篇文章已经讲了Java内存模型和与其三个特性：原子性、可见性、有序性。但事实上，为了提升程序的执行性能，**编译器** 和 **处理器** 常常会对程序指令序列进行 **重排序**。

##### 重排序分为以下几种：

- 编译器优化重排序
- 处理器重排序
  - 指令级并行重排序
  - 内存系统重排序



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、屏障指令</h3>

![fence](https://i.loli.net/2018/12/19/5c19f01f8d3da.jpg)

> 内存屏障（Memory Barrier，或称为内存栅栏，Memory Fence）是一种CPU指令，用于控制特定条件下的重排序和内存可见性问题。Java编译器也会根据内存屏障的规则在一定程度地禁止重排序。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、as-if-serial 语句</h3>

> 重排序也不能毫无规则，否则语义就变得不可读， **as-if-serial语句** 给重排序戴上紧箍咒，起到约束作用。

**as-if-serial语句** 规定重排序要满足以下两个规则：

- 在单线程环境下不能改变程序执行的结果；
- 存在数据依赖关系代码（指令）片段的不允许重排序。

比如下面的代码：

```java
int a = 1; // ①
int b = 2; // ②
int c = a + b; // 依赖于 ① 和 ②
return c;
```

可能会被优化成：

```java
int b = 2; // ②
int a = 1; // ①
int c = a + b; // 依赖于 ① 和 ②
return c;
```

上述的重排序既没有改变单线程下程序运行的结果，又没有对存在依赖关系的指令进行重排序。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">四、happens-before 规则</h3>

> 产生的背景是为了确保多线程操作下具有内存可见性。
>
> 如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须要存在happens-before关系。换句话说，操作1 happens-before 操作2，那么操作1的结果是对操作2可见的。
>
> 这里提到的两个操作既可以是在一个线程之内，也可以是在不同线程之间。



#### 规则：

- **1. 程序顺序规则**：一个线程中的每个操作，happens-before 于该线程中的任意后续操作
- **2. 监视器锁规则**：对一个锁的解锁，happens-before 于随后对这个锁的加锁
- **3. volatile变量规则**：对一个volatile域的写，happens-before 于任意后续对这个volatile域的读
- **4. 传递性**：如果A happens-before B，且B happens-before C，那么A happens-before C
- **5. start规则**：如果线程A执行操作ThreadB.start()（启动线程B），那么A线程的ThreadB.start()操作happens-before于线程B中的任意操作
- **6. join规则**：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回。



### 总结：

**as-if-serial** 语句保证单线程环境下不能改变程序执行的结果，**happens-before** 规则保证多线程环境下不能改变程序执行的结果。