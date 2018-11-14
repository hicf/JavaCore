<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">Java内存模型之：synchronized</h3>



#### 本文关注点
* **Class文件与对象**
* **synchronized 三种应用方式**
* **synchronized 底层语义原理**
* **JVM对synchronized的优化**
* **synchronized关键点**

---
#### 一、Class文件与对象
> 对象头
> ![32位JVM的对象头](https://upload-images.jianshu.io/upload_images/11476758-4907bdaeaac7f527.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---
#### 二、synchronized 三种应用方式
> 用于实例对象方法（对象是锁）
> 用于静态方法（类是锁）
> 用于同步代码块（同步代码块是锁）

---
#### 三、synchronized 底层语义原理
> * 锁：就在对象头中
> * Java 虚拟机中的同步(Synchronization)基于 **进入enter** 和 **退出exit**  **监视器(Monitor)对象** 实现， 无论是显式同步(有明确的 monitorenter 和 monitorexit 指令,即同步代码块)还是隐式同步都是如此。在 Java 语言中，同步用的最多的地方可能是被 **synchronized** 修饰的同步方法。**同步方法 并不是由 monitorenter 和 monitorexit 指令来实现同步的，而是由方法调用指令读取运行时常量池中方法的 ACC_SYNCHRONIZED 标志来隐式实现的，关于这点，稍后详细分析**。下面先来了解一个概念Java对象头，这对深入理解synchronized实现原理非常关键。

##### 2.1、对象加锁：
> 使用 **monitorenter** 和 **monitorexit** 指令分别获取控制权和释放控制权。
> ![monitor](https://upload-images.jianshu.io/upload_images/11476758-633049c76385f4d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> ![获取锁与释放锁](https://upload-images.jianshu.io/upload_images/11476758-9d8c25afef7a8ea6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 2.1、方法加锁：
> 方法级的同步是隐式，**即无需通过字节码指令来控制的，它实现在方法调用和返回操作之中**。JVM可以从方法常量池中的方法表结构(method_info Structure) 中的 **ACC_SYNCHRONIZED** 访问标志区分一个方法是否同步方法。**当方法调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先持有monitor（虚拟机规范中用的是监视器一词）， 然后再执行方法，最后再方法完成(无论是正常完成还是非正常完成)时释放monitor**。在方法执行期间，执行线程持有了monitor，其他任何线程都无法再获得同一个monitor。如果一个同步方法执行期间抛 出了异常，并且在方法内部无法处理此异常，那这个同步方法所持有的monitor将在异常抛到同步方法之外时自动释放。下面我们看看字节码层面如何实现：

---
####  四、JVM对synchronized的优化
> 背景：**Java 6**之后，为了减少获得锁和释放锁所带来的性能消耗，引入了 **偏向锁** 和 **轻量级锁**。
> 锁的状态：**无锁状态、偏向锁、轻量级锁、重量级锁**。
> 锁从 **偏向锁 -> 轻量级锁 -> 重量级锁** 的锁升级是单向的、不可逆的，没有 ~~锁降级~~ 这一说。

##### 4.1、偏向锁
> 因此为了减少同一线程获取锁(会涉及到一些CAS操作,耗时)的代价而引入偏向锁。偏向锁的核心思想是，如果一个线程获得了锁，那么锁就进入偏向模式，此时Mark Word 的结构也变为偏向锁结构，当这个线程再次请求锁时，无需再做任何同步操作，即获取锁的过程，这样就省去了大量有关锁申请的操作，从而也就提供程序的性能。

##### 4.2、轻量级锁
> 轻量级锁所适应的场景是 **线程交替执行同步块** 的场合，如果存在 **同一时刻访问同一锁** 的场合，就会导致轻量级锁膨胀为重量级锁。

##### 4.3、`★★★★★`自旋锁
> 自旋锁是乐观的一种表现，乐观的认为很大概率是能够获得锁的。（根据阿里巴巴Java开发规范：如果每次访问冲突的概率小于 **20%**，推荐使用乐观锁，否则使用悲观锁。乐观锁的重试次数不得小于**3次**。）

##### 4.4、重入锁
> 从互斥锁的设计上来说，当一个线程试图操作一个由其他线程持有的对象锁的临界资源时，将会处于阻塞状态，但当一个线程再次请求自己持有对象锁的临界区的资源时，这种情况属于 ***重入锁**，请求将会成功，在java中synchronized是基于原子性的内部锁机制，是可重入的，因此在一个线程调用synchronized方法的同时在其方法体内部调用该对象另一个synchronized方法，也就是说一个线程得到一个对象锁后再次请求该对象锁，是允许的，这就是synchronized的可重入性。

##### 4.5、`★★★★★`锁消除
> 消除锁是虚拟机另外一种锁的优化，这种优化更彻底，Java虚拟机在JIT编译时(可以简单理解为当某段代码即将第一次被执行时进行编译，又称即时编译)，通过对运行上下文的扫描，去除不可能存在共享资源竞争的锁，通过这种方式消除没有必要的锁，可以节省毫无意义的请求锁时间，如下StringBuffer的append是一个同步方法，但是在add方法中的StringBuffer属于一个局部变量，并且不会被其他线程所使用，因此StringBuffer不可能存在共享资源竞争的情景，JVM会自动将其锁消除。

---
####  五、synchronized关键点
> 中断与synchronized：
> 事实上线程的中断操作对于正在等待获取的锁对象的synchronized方法或者代码块并不起作用，也就是对于synchronized来说，**如果一个线程在等待锁，那么结果只有两种，要么它获得这把锁继续执行，要么它就保存等待，即使调用中断线程的方法，也不会生效。**


> 参考资料：
> [深入理解Java并发之synchronized实现原理](https://blog.csdn.net/javazejian/article/details/72828483)