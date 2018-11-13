<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">零、GC垃圾收集器</h3>

![GC垃圾收集器](https://upload-images.jianshu.io/upload_images/11476758-a1cf2fa524ae40cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* Serial 串行收集器（年轻代：Serial New复制，老年代：Serial Old标记-整理）

- Parallel 并行收集器（年轻代：Parallel New复制、Parallel Scavenge（低延时）复制，老年代：Parallel Old标记-整理）
- CMS（Concurrent Mark Sweep）并发-标记清除收集器（老年代：标记-清除，需要Serial Old）
- G1（Garbage First）收集器（Region区域，收集大的垃圾）
- Epsilon（No-op 无操作）垃圾收集器
- ZGC 可扩展、低延时的垃圾收集器

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、Serial 串行垃圾收集器</h3>

![Serial 串行垃圾收集器]()

>  **特点**：单线程
> **垃圾收集算法**：分代垃圾回收算法，新生代采用 **复制算法**，老年代采用 **标记-整理算法** 。
> **缺点**：延迟大，收集垃圾时首先 **暂停所有工作线程Stop-The-World**(简称STW)，然后单线程地进行收集垃圾。
> **适用于**：对GC延迟不敏感的客户端。另外，如果很多个jvm运行在同一台机器上时，也适合使用这种算法。
> **开启参数**：`-XX:+UseSerialGC`

##### Serial Old老年代收集器

> 这也是一个单线程的垃圾收集器，作用于老年代。
>
> 作用：搭配Parallel Scavenge使用或作CMS老年代垃圾回收的备用方案，只是CMS失败时才用Serial Old。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、Parallel 并行垃圾收集器</h3>

![Parallel 并行垃圾收集器]()

>  Parallel 并行垃圾收集器Parallel New、Parallel Scavenge、Parallel Old

#### 2.1、Parallel New

> 特点：Parallel New是多线程（进行年轻代垃圾收集，是Serial多线程版本。当主机有N个 CPU 时，默认会使用N个垃圾收集线程。使用`-XX:ParallelGCThreads=<线程数目>`控制线程数），ParOld默认单线程。
> 垃圾收集算法：**标记-整理算法**（另外，使用`-XX:+UseParallelOldGC`命令行选项可以让新生代和老年代同时使用多线程并行垃圾收集。在老年代，ParallelOld是一种 标记-整理 算法，会在标记后将存活对象搬运到一起来防止出现内存碎片。）
> 缺点：注意，在单核主机上，是不会使用Parallel GC的，在2核主机上，Parallel GC 可能和默认的 GC 收集器工作效率相同，而在大于2核的主机上，Parallel GC 一般会有更好的表现。当然，一切还是需要以实际测量为准。GC延迟大。
> 适用于：Parallel GC适合于需要高吞吐量而对暂停时间不敏感的场合，比如批处理任务。
>
> 开启参数：`-XX:+UseParallelGC`

#### 2.2、Parallel Scavenge

> **Parallel Scavenge和Parallel New 都是作用于新生代的垃圾收集器**，区别在于，Parallel Scavenge牺牲垃圾回收的吞吐量，达到低延时的垃圾回收时，从而使得应用线程的等待状态大大降低。同时垃圾收集更加频繁。
> **Parallel Scavenge** 的目的是尽可能缩短垃圾收集时用户线程的停顿时间。

#### 2.3、Parallel Old

> Parallel Old是Parallel Scavenge的老年代版本，多线程收集器。起始于jdk1.6。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、CMS 并发标记-清除收集器</h3>

> CMS全称ConcurrentMarkSweep。
>
> 产品定位：以 **最短回收停顿时间** 为目标的收集器。
> 背景：目前基于B/S系统服务尤其注重 **响应速度**，希望系统停顿时间短。
> 特点：最小化响应时间，GC线程在某些垃圾收集步骤中可以和应用线程并行进行。
> 垃圾收集算法：TODO

#### CMS垃圾收集的周期（3标记1清除）：

> - **初始标记（CMS-initial-mark）**：标记roots能直接引用到的对象；
> - **并发标记（CMS-concurrent-mark）**：进行GC root跟踪
> - **重新标记（CMS-remark）**：修正并发标记阶段因用户线程运行而导致的变动
> - **并发清除（CMS-concurrent-sweep）**：进行并发清除工作

#### CMS的优点：

> 1）、初始标记和重新标记会导致 Stop-The-World。`由于最耗时的并发标记和并发清除都可以和用户程序同时进行，所以其实可以认为 GC 和用户程序是同时进行的。`
> 2）、`CMS的一个劣势是对CPU资源比较敏感，不过现代的后端系统通常是重IO的，所以个人感觉影响并不会太大。`另外一个问题是，由于 GC 和用户程序同时进行，可能会有部分新产生的垃圾无法被直接回收，需要等到下一次 GC 时再回收。也是由于在垃圾收集阶段用户线程还需要运行，那也就还需要预留有足够的内存空间给用户线程使用，因此 **CMS收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，需要预留一部分空间提供并发收集时的程序运作使用** 。使用`-XX:CMSInitiatingOccupancyFraction`可以定义老年代使用了多少空间后就开始进行 GC，默认为68%。
> 3）、 CMS的在新生代与 Parallel 一样使用 **并行复制收集器**，而在老年代使用 **标记-清除 算法**。`在老年代，由于不会进行整理操作，会导致空间碎片的出现，如果空间碎片过多，则需要使用Serial Old进行一次老年代垃圾回收，为了防止该情况出现，可以设置若干次CMS GC以后进行一次内存整理；同时，由于和应用线程并行的进行垃圾回收，所以需要在内存耗尽前就开始垃圾收集，否则可能导致应用无内存可用。`

#### 启用参数

> 启用CMS参数`-XX:+UseConcMarkSweepGC`。
>
> CMS默认回收线程数目是 `ParallelGCThreads + 3)/4` ，`ParallelGCThreads` 是年轻代并行收集线程数目，可以通过`-XX:ParallelCMSThreads=2` 来设置。
>
> 可以使用`-XX:+UseCMSCompactAtFullCollection` 开启CMS阶段整理内存。
>
> 可以使用`-XX:+CMSFullGCsBeforeCompaction=<次数>` 设置每多少次CMS后进行一次内存整理。
>
> 老年代进行CMS时的内存消耗百分比，默认为68，可以使用`-XX:CMSInitiatingOccupancyFraction=百分比`设置。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">四、G1收集器(Garbage First）</h3>

> 前言：G1是具有全新、划时代意义的、比较前沿的、流行广泛的垃圾收集器，可以替换JDK1.5的CMS。
> 垃圾收集算法：分代收集
> 优点：
> - 并行与并发：G1收集器通过 **并发** 方式让Java工作程序继续运行。
> - 分代收集：G1保留着分代收集的概念，但它不像CMS收集器那样需要Serial Old（在收集老年代时）配合使用，**G1可以独立不依赖其他收集器工作**。
> - 空间整合：CMS老年代是标记-清除，**G1是标记-整理，从局部（两个Region之间）来看是基于复制算法**。不会像CMS那样产生碎片空间，收集后可提供规整的内存空间。
> - 可预测停顿：CMS和G1都是追求垃圾收集时间的低延迟，相比而言，**G1还能预测停顿时间模型**。

#### 特点：

> 其他收集器的收集范围是整个年轻代或老年代，而G1则不是这样的了。使用G1时，Java堆的内存分布差别很大，它将整个Java堆划分为多个大大小小相等的独立 **区域（Region）**，虽然像其他收集器保留着年轻代、老年代的分代概念，但是新生代和老年代不再是物理隔离的了，它们都是逻辑上连续的多个Region的集合。
> 优先回收策略：**优先回收最大的Region，保证了有限时间可以仅可能提供工作效率。**

#### G1垃圾收集的周期（3标记1回收）：

> - **初始标记**（Initial Marking）
> - **并发标记**（Concurrent Marking）
> - **最终标记**（Final Marking）
> - **筛选回收**（Live Data Counting and Evacuation）

#### 区域Region：

>  区域Region大小是 2 的次幂，范围是 1 MB 到 32 MB 之间。目的是根据最小的 Java 堆大小划分出约 1024/2048 个区域。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">五、ZGC 收集器</h3>

> 此款垃圾收集器是jdk11新添加的垃圾收集器。它是可扩展、低延时的垃圾的垃圾收集器，支持TB级的内存区域，最大堆内存4TB，延时不超过 10ms。

#### 目标：

> - 每次GC STW的时间不超过10ms
> - 能够处理从几百M到几T的JAVA堆
> - 与G1相比，吞吐量下降不超过15%
> - 为未来的GC功能和优化利用 **有色对象指针(colored oops)** 和 **加载屏障(load barriers)** 奠定基础
> - 初始支持Linux/x64

#### 特点：

> - 并发
> - 基于Region
> - 标记-整理
> - NUMA感知
> - 使用有色对象指针(
> - 使用加载屏障
> - 仅root扫描时出现延时STW（Stop-The-World），因此GC暂停时间不会随堆的大小而增加。

#### 垃圾回收步骤：

> - **递归标记**
> - **迁移对象** （relocate）



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">六、Epsilon（No-op 无操作）垃圾收集器</h3>

> Java 11还加入了一个比较特殊的垃圾回收器——Epsilon，该垃圾收集器被称为“no-op”收集器，将处理内存分配而不实施任何实际的内存回收机制。 也就是说，这是一款不做垃圾回收的垃圾回收器。它的作用就是与“做垃圾回收的收集器”做对比使用。