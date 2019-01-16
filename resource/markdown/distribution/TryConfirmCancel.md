<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">TCC事务补偿机制（柔性事务方案）</h3>

**TCC** 是 *Try Confirm-Cancel* 的缩写，其本质上还是2PC（两阶段提交协议）。它不像具有[ACID特性](https://github.com/about-cloud/JavaCore)的数据库事务那样刚性 ，TCC是 **柔性事务方案**。为什么称为柔性事务方案呢？因为它不直接依赖于资源管理（RM），回想一下[前面的XA事务](https://github.com/about-cloud/JavaCore)，其中就直接依赖于资源管理器，而 TCC 是在业务层面处理的，业务依赖于何种资源管理器、依赖多少资源管理器，TCC 中是不需要担心，所以它的方式更为弹性。

![TCC](https://i.loli.net/2019/01/16/5c3f4b5b2e21d.png)



一个完整的TCC模式有以下组成部分：

1、业务应用：直接使用的应用；

2、被调用的服务：一般是多个，是主业务应用被调用的服务，以此来获得资源；

3、事务协调器：开启事务、注册事务，协调事务内的服务提交/回滚活动，从而达到全局事务的一致性。



#### 第一阶段：Try阶段：

检查和预留资源阶段，和2PC的第一阶段相似 -- 占有资源。



#### 第二阶段：Confirm/Cancel阶段：

Confirm表示确认执行业务操作，Cancel表示取消执行业务操作。



#### TCC 这种入侵业务而不接触资源管理器的优缺点如下：

**优点:** 资源管理的粒子度可自控，在资源层面可以做到很好的控制；

**缺点:** 入侵业务代码，耦合度增加，每个被调用的服务都要重复实现Try、Confirm、Cancel接口代码，改造成本高。



参考资料：

阿里技术