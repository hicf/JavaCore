<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">线程的状态</h3>

![线程状态图](https://i.loli.net/2018/12/18/5c18688584a91.png)

#### 1、新建状态(New)

> 万事万物都不是凭空出现的，线程也一样，它被创建后的状态称为 **新建** 状态。

比如：

```java
Thread thread = new Thread();
```



#### 2、可运行状态(Runable)

> 线程被创建后是不能使用的，就是让用户在此期间设置一些属性，

比如：

```java
// 设置类加载器
thread.setContextClassLoader(System.class.getClassLoader());
// 设置线程名称
thread.setName("商品服务-product-service");
// 是否为守护线程/用户线程
thread.setDaemon(false);
// 设置线程优先级
thread.setPriority(5);
```

通过 `thread.start()` 方法开启线程，开启后意味着该线程 **“能够”** 运行，并不意味着一定会运行，因为它要抢占资源，获取CPU的使用权后，才能运行。所以此状态称为 **可运行状态**。从上图中可以看出，不仅通过 `start()` 启动一个线程后可以进入 Runnable 状态，还可以通过其他方式到达 Runnable 状态。 



#### 3、运行状态(Running)

> 线程通过努力，获得了CPU的使用权，就会进入执行程序，此时状态被称为 **运行状态**。



#### 4、阻塞状态(BLOCKED)

> 多线程抢占CPU资源，同一时刻仅有一个线程进入临界区，为保证对资源访问的线程安全，同一时刻仅有一个线程进入 synchronized 同步块，而其他未获得访问权的线程将进入 **阻塞状态** 。

**等待阻塞**：通过调用线程的wait()方法，让线程等待某工作的完成。

**同步阻塞**：线程在获取synchronized同步锁失败(因为锁被其它线程所占用)，它会进入同步阻塞状态。

**其他阻塞**：通过调用线程的sleep()或join()或发出了I/O请求时，线程会进入到阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。



#### 5、睡眠状态 TIMED_WAITING(sleeping) 

> 通过调用对象的wait(time)方法或调用线程的sleep(time)/join(time)，等待/睡眠指定的时间，此时该线程会进入TIMED_WAITING(sleeping) 状态，直接时间已到，会进入Runnable状态，重新抢占CPU资源。



#### 6、等待状态 WAITING

> 通过调用对象的wait()方法，让抢占资源的线程等待某工作的完成，或主动join()其他线程，让当前线程释放资源等待被join的线程完成工作，而该线程将进入 **等待状态** 。



#### 7、死亡状态(Dead)

> 线程执行完了或者因异常退出了run()方法，该线程结束生命周期。

