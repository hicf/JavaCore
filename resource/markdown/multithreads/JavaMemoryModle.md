<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">Java内存模型三大特性：原子性、可见性、有序性(顺序性)</h3>



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、原子性</h3>

> 编程界的 **原子性** 都是共性的，只不过表现的载体不一样罢了。
>
> 一个操作中有多个子操作，宏观来看，该操作是不可中断的，要么全部执行成功要么全部执行失败，该操作与子操作有着“同生共死”的感觉，这就是 **原子性**。

举个例子（仅仅是辅助理解原子性）：

```java
// 下面有三个操作
a = 1;
b = 2;
c = 3;

// 假设将这三个操作封装成一个大的操作A
A {
    a = 1;
	b = 2;
	c = 3;
}
```

如果A操作时原子操作，那么A中的所有操作都会执行成功（或都会执行失败），原子性强调的整体观。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、可见性</h3>

> 此处的 **可见性** 指的是 **内存可见性**。它是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够 **立即** 看得到修改后的最新值。你可以理解成，每时每刻看到的值都是最新值。

在Java中，可以使用 **volatile** 保证多线程下的 **内存可见性**。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、有序性</h3>

> 即程序执行的顺序按照代码的先后顺序执行。

