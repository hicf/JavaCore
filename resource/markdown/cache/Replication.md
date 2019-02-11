<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">主从复制</h3>

**数据复制的意义:** 1、读写分离，降低单节点的读写压力；2、容灾转移，单机出现问题，从节点接替。

Redis 数据复制是单向的，而且一个节点从属一个master节点。



设置主从复制方式一: redis实例启动前，在配置文件中设置 `slaveof host port`

设置主从复制方式二: redis实例启动时，使用 `redis-server` 加上 `--slaveof host port`

设置主从复制方式三:redis实例启动后，在命令行设置 `slaveof host port`

断开主从复制:在从节点命令行执行 `slaveof no one`



#### 一主一从: 

![一主一从](https://i.loli.net/2019/02/11/5c612b5f58ffc.png)

常规的主从配置，一个主节点配置一个从节点。

作主备容灾转移时，主节点负责读写，从节点负责复制数据，一旦主节点宕机，从节点晋升为主节点，接替读写工作；

做读写分离时，主节点只写，从节点只读。

注意，节点最好要开启持久化，否则Redis实例宕机重启后，数据变空。



#### 一主多从:

![一主多从](https://i.loli.net/2019/02/11/5c612ba97b585.png)

根据“二八原则”，大部分场景下都是在读操作，通常给主节点配置多个从节点，主节点只写，从节点只读，从节点间分摊压力。



#### 树状主从:

![树状主从](https://i.loli.net/2019/02/11/5c612bbf22aea.png)

但是从节点过多，主节点的IO就越大（主要是IO中的O），这时就会出现IO堵塞。使用树状主从结构，每个主节点的从节点数量就变少，这样主节点的IO输出压力就会变小，非叶子节点的主节点层层分担压力，从而降低根节点的压力。

(为了能够校正数据，不推荐将从节点设置为只读模式 ~~slave-read-only=yes~~)



---

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">复制原理</h3>

![Redis注册复制原理](https://i.loli.net/2019/02/11/5c6147b3de702.png)

