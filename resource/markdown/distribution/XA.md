<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">X/Open DTP 与 XA事务</h3>

**X/Open** 是一个欧洲基金会的公司名称，中文名称国际联盟有限公司。



**X/Open DTP** 全称 *X/Open Distributed Transaction Processing Reference Model* ，中文名称 **X/Open 分布式事务处理参考模型**，是X/Open 基金会规范的一个分布式事务处理模型，当然这个模型也是概念模型。

![X/Open DTP](https://i.loli.net/2019/01/15/5c3dcc685cfb5.png)

**X/Open DTP** 中定义了三个组件：

- **AP（Application Program）：**参与分布式事务中的应用程序，构成了所需的一个或多个操作；
- **RM（Resource Manager）:**资源管理器，类似的RDBMS（Relational Database Management System）关系数据库管理系统，主要作用还是管理资源，那就意味着，事务想要获得资源，必须要经过RM管理。其中RM自身支持事务（比如InnoDB），RM能够根据将全局（分布式）事务标识定位到自己内部的对应事务。
- **TM（Transaction Manager）:**事务管理器，负责协调和管理AP和RM之间的分布式事务完整性。主要的工作是提供AP注册全局事务的接口，颁发全局事务标识（GTID之类 的），存储/管理全局事务的内容和决策并指挥RM做commit/rollback。



其中在DTP定了以下几个概念：

**事务:** 一个事务是一个完整的工作单元，由多个独立的计算任务组成，这多个任务在逻辑上是原子的。

**全局事务:** 对于一次性操作多个资源管理器的事务，就是全局事务

**分支事务:** 在全局事务中，某一个资源管理器有自己独立的任务，这些任务的集合作为这个资源管理器的分支任务

**控制线程:** 用来表示一个工作线程，主要是关联AP,TM,RM三者的一个线程，也就是事务上下文环境。简单的说，就是需要标识一个全局事务以及分支事务的关系。



---

**XA**，全称 *eXtended Architecture*，

