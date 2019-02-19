比如某个应用需要定时跑一些任务，为了保证高可用，做了三个节点的集群，后来发现集群中每个节点都跑了重复的任务。

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、基于数据库实现分布式锁</h3>

首先创建一张表，用于存储锁标识的记录:

```mysql
CREATE TABLE IF NOT EXISTS distribution_lock(
  id BIGINT(20) UNSIGNED NOT NULL COMMENT '分布式锁id',
  lock_name VARCHAR(50) DEFAULT '' COMMENT '分布式锁的名称',
  node_number TINYINT(1) NOT NULL COMMENT '集群下节点的编号，用于重入锁',
  gmt_create DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '行记录创建时间',
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='存储分布式锁的表';
```



**获取锁:**

```mysql
INSERT INTO distribution_lock(id, lock_name, node_number) VALUES(1001, 'order_task', 1);
```

插入成功表示获取锁，否是未获取锁；



**获取重入锁:**

```mysql
SELECT
  t0.id,
  t0.lock_name,
  t0.gmt_create 
FROM distribution_lock t0
WHERE t0.id = 1001 AND t0.node_number = 1;
```



**检查锁超时:**

```mysql
SELECT
  t0.id,
  t0.lock_name,
  t0.gmt_create ,
  TIMESTAMPDIFF(SECOND, t0.gmt_create, NOW()) 'duration_time'
FROM distribution_lock t0
WHERE t0.id = 1001 AND t0.node_number != 2;
```



**释放锁:**

```mysql
DELETE FROM distribution_lock WHERE id = 1001;
```



> 上面是用 `INSERT` + `SELECT ` + `DELETE` 来实现的，当然也可以使用 `UPDATE` + `SELECT` 实现（注意初始化锁标识的行记录）。

因为 `id` 设置为 主键，所以具有唯一约束属性，同一时刻只能被一个线程获取锁🔐。



**注意问题:**

1. 单点故障: 数据库存在单点故障的风险，建议做成高可用版本；
2. 重入性: 也就是重入锁，可以精确到线程级别；
3. 锁失效机制: 如果一台服务获得锁后，再没有主动释放锁的情况下，其他服务也就无法获取锁。解决方法是获取锁时，判断锁是否超时；
4. 非阻塞性: 比如一个定时任务每月跑一次，恰巧此时被其他节点服务占用锁，导致该节点获取锁失败，解决方法是循环多次获取锁；
5. 性能: 基于数据库的性能比较低，不适合高并发场景。



---

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、基于Redis实现分布式锁</h3>

**获取锁**

```shell
127.0.0.1:6379> SETNX lock:1001 192.168.2.104
```

巧用 `SETNX` 命令，其中 `value` 设置唯一标识，比如 `ip+线程id+时间戳` 、 `UUID`诸如此类，如果返回 `1` 表示获取锁成功，否则获取锁失败；



**设置锁超时时间**

```shell
127.0.0.1:6379> expire lock:1001 30
```

如果返回 `1` 表示设置锁超时成功，否则失败；



**锁过期检查**

```shell
127.0.0.1:6379> GET lock:1001
```

如果获得的 `value` 和之前设置的 `value` 说明锁有效，提交事务；否则锁过期，业务回滚；



**释放锁**

```shell
127.0.0.1:6379> DEL lock:1001
```



注意: 防止误删 `key` ，使用以下伪代码演示:

```java

```





---

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、基于Zookeeper实现分布式锁</h3>

**获取锁**





**释放锁**

