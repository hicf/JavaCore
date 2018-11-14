<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、重排序</h3>

> 为了提升程序的执行性能，**编译器** 和 **处理器** 常常会对程序指令序列进行 **重排序**。

##### 重排序分为以下几种：

- 编译器优化重排序
- 处理器重排序
  - 指令级并行重排序
  - 内存系统重排序



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、屏障指令</h3>

> 内存屏障（Memory Barrier，或有时叫做内存栅栏，Memory Fence）是一种CPU指令，用于控制特定条件下的重排序和内存可见性问题。Java编译器也会根据内存屏障的规则禁止重排序。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、as-if-serial语句</h3>

> **as-if-serial语句** 给重排序戴上紧箍咒，起到约束作用。它规定不管怎么重排序，**（单线程）程序的执行结果不能被改变**。编译器、运行时和处理器都必须遵守as-if-serial语义。 为了遵守as-if-serial语义，编译器和处理器不会对存在数据依赖关系的操作做重排序。as-if-serial语义把单线程程序保护了起来，as-if-serial语义使单线程程序员无需担心重排序会干扰他们，也无需担心内存可见性问题。

比如：

```java
int a = 1;
int b = 1 + a;
int c = 2;
int d = b + c;
// 有可能优化成以下情况
int a = 1;
int c = 2;
int b = 1 + a;
int d = b + c;
```



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">四、happens-before规则</h3>

> 如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须要存在happens-before关系。**这里提到的两个操作既可以是在一个线程之内，也可以是在不同线程之间。**

![happens-before规则](https://upload-images.jianshu.io/upload_images/11476758-07e4c5ce8a9cf95b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 规则：

- **1. 程序顺序规则**：一个线程中的每个操作，happens-before 于该线程中的任意后续操作
- **2. 监视器锁规则**：对一个锁的解锁，happens-before 于随后对这个锁的加锁
- **3. volatile变量规则**：对一个volatile域的写，happens-before 于任意后续对这个volatile域的读
- **4. 传递性**：如果A happens-before B，且B happens-before C，那么A happens-before C
- **5. start规则**：如果线程A执行操作ThreadB.start()（启动线程B），那么A线程的ThreadB.start()操作happens-before于线程B中的任意操作
- **6. join规则**：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回。