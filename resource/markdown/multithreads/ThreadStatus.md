<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">线程的状态</h3>

![线程状态图](https://images2015.cnblogs.com/blog/716271/201703/716271-20170320112245721-1831918220.jpg)

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

通过 `thread.start()` 方法开启线程，开启后意味着该线程 **“能够”** 运行，并不意味着一定会运行，因为它要抢占资源，获取CPU的使用权后，才能运行。所以此状态称为 **可运行状态**。



#### 3、运行状态(Running)

> 线程通过努力，获得了CPU的使用权，就会进入执行程序，此时状态被称为 **运行状态**。



#### 4、阻塞状态(Blocked)

> 一个线程通过 **时间片轮询** 分配的资源是有限的，也绝不能允许长时间占用资源，这是 **被迫式** 的放弃资源，然后进入 **阻塞状态**。也有正在运行中的线程自己主动让出CPU给其他线程使用，自己进入 **阻塞状态**。阻塞状态又分为以下三种：

**等待阻塞**：通过调用线程的wait()方法，让线程等待某工作的完成。

**同步阻塞**：线程在获取synchronized同步锁失败(因为锁被其它线程所占用)，它会进入同步阻塞状态。

**其他阻塞**：通过调用线程的sleep()或join()或发出了I/O请求时，线程会进入到阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。



#### 5、死亡状态(Dead)

> 线程执行完了或者因异常退出了run()方法，该线程结束生命周期。

