<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、CAP原则</h3>

**CAP原则** (又称*CAP*定理) 是指一个分布式系统中，*Consistency*（一致性）、 *Availability*（可用性）、*Partition tolerance*（分区容错性），三者不可兼得。

![CAP原则]()

#### 1、一致性（Consistency）

> 在分布式系统中的所有数据备份，在同一时刻是否同样的值。（等同于所有节点访问同一份最新的数据副本）



#### 2、可用性（Availability）

> 在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。（对数据更新具备高可用性）



#### 3、分区容错性（Partition tolerance）

> 以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择。



---

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、BASE理论</h3>

**BASE** 基本可用（Basically Available）、软状态（Soft state）、最终一致（Eventually consistent）三个短语的缩写。BASE是对CAP中一致性和可用性权衡的结果，其来源于对大规模互联网系统分布式实践的结论，是基于CAP定理逐步演化而来的，其核心思想是即使无法做到强一致性（Strong consistency），但每个应用都可以根据自身的业务特点，采用适当的方式来使系统达到最终一致性（Eventual consistency）。

#### 1、基本可用（Basically Available）



#### 2、软状态（Soft state）



#### 3、最终一致（Eventually consistent）