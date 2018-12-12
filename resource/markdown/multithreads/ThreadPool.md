<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">ThreadPoolExecutor线程池</h3>

线程的创建和销毁都会消耗大量资源，就好像公司每天上午9点工作时就招进一批员工，晚上6点干完活就辞退一批员工，这都会销毁公司大量资源。所以合理利用 “池” 中固定、稳定的线程是非常有必要的。

### 扩展关系

![ThreadPoolExecutor继承关系](https://i.loli.net/2018/12/12/5c1070e4ab3ca.png)

#### ThreadPoolExecutor 构造方法

ThreadPoolExecutor 共有四个构造方法：

```java
ThreadPoolExecutor(int, int, long, TimeUnit, BlockingQueue<Runnable>)

ThreadPoolExecutor(int, int, long, TimeUnit, BlockingQueue<Runnable>, ThreadFactory)

ThreadPoolExecutor(int, int, long, TimeUnit, BlockingQueue<Runnable>, RejectedExecutionHandler)

ThreadPoolExecutor(int, int, long, TimeUnit, BlockingQueue<Runnable>, ThreadFactory, RejectedExecutionHandler)
```

以最多参数的构造方法为例进行分析：

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    // 核心线程池不能小于0
    if (corePoolSize < 0 ||
        // 最大池大小不能小于等于0
        maximumPoolSize <= 0 ||
        // 最大池大小不能小于核心池大小
        maximumPoolSize < corePoolSize ||
        // 存活时间不能小于0
        keepAliveTime < 0)
        // 否则将会抛出 IllegalArgumentException 非法参数异常
        throw new IllegalArgumentException();
    // 工作队列、线程工厂、拒绝执行的处理策略都不能为空，否则将会排除NPE空指针异常
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?
            null :
            AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

#### ①参数 corePoolSize 核心线程池大小：

>  线程池中一直会存活该大小的线程数，即使是没有工作(任务)需要执行。除非设置 `allowCoreThreadTimeOut` 为 `true` ，线程池中的核心线程会超时关闭。



#### ②参数 maximumPoolSize 线程池最大大小：

> 线程池最大允许同时存活的线程的大小。



#### ③参数 keepAliveTime 线程空闲时间：

> 当线程池中的线程空闲时间达到 `keepAliveTime` 时，线程会被销毁，仅保留 `corePoolSize` 大小线程，如果`allowCoreThreadTimeOut` 为 `true` ，则线程池中包含核心线程在内空闲线程都会被销毁。



#### ④参数 unit 时间单位：

> `keepAliveTime` 的时间单位（枚举类型）`TimeUnit` ，其可选单位有 `TimeUnit.DAYS` 天、`TimeUnit.HOURS` 小时、`TimeUnit.MINUTES` 分钟、`TimeUnit.SECONDS`秒、`TimeUnitMILLISECONDS.`毫秒、`TimeUnit.MICROSECONDS`微秒、`TimeUnit.NANOSECONDS`纳秒，常用的是秒。



#### ⑤参数 workQueue 任务队列(工作队列、缓存队列)：

> 当任务所需的线程数达到核心线程数 `corePoolSize ` 时，新任务会放在工作队列中排队（缓存）等待执行。如果任务所需的线程数达到核心线程数 `corePoolSize ` 时，并且工作队列已满时，并且线程池最大大小 `maximumPoolSize ` 大于 核心线程池大小 `corePoolSize`  时，才会创建新的线程去处理任务。
>
> *经常有个错觉，~~认为当任务所需的线程数达到线程池最大线程数 `maximumPoolSize ` 时，新任务才会进入工作队列~~，这是不对的。



#### ⑥参数 threadFactory 线程工厂：

> 用于创建线程池中线程的工厂。创建线程时，经常会给这一批具有处理相同类型任务的线程命名和线程工厂命名（*线程工厂命名是指给poolName线程池命名，作为线程名称的前缀prefix），以此来标识线程的用处，在分析程序执行信息或排查程序异常问题时，非常有帮助。



#### ⑦参数 handler 拒绝处理策略：

> 线程池拒绝处理任务有一下两者情况：
>
> 1）、当线程池中的线程数达到核心池大小时，并且任务队列已满，会使用拒绝处理策略；（此种情况就像工作台上的工作量积累已满，又没有足够的人员去处理，那么我们就可以拒绝处理新任务。）
>
> 2）、当调用shutdown() 方法后，会等待线程池中正在执行的任务执行完毕，然后再关闭线程池，如果在调用 shutdown() 方法后（还未真正关闭时），紧接着又有新的任务提交时，会使用拒绝处理策略。（此种情况就像工作到了下班时间，工作人员还在忙着手头上剩余的工作，如果此时又新任务提交，那么我们就可以拒绝处理新任务。）
>
> （当然也可以不拒绝）

**拒绝处理策略：**

| 拒绝处理策略        | 是否默认 | 描述                                                 |
| ------------------- | -------- | ---------------------------------------------------- |
| AbortPolicy         | 是       | 丢弃任务，抛RejectedExecutionException异常；         |
| ~~DiscardPolicy~~   |          | 直接忽视，也不会抛出异常，不推荐使用；               |
| DiscardOldestPolicy |          | 从任务队列中踢出最先进入队列（最后一个执行）的任务； |
| CallerRunsPolicy    |          | 执行新任务。                                         |

