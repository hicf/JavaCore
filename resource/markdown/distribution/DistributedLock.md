比如某个应用需要定时跑一些任务，为了保证高可用，做了三个节点的集群，后来发现集群中每个节点都跑了重复的任务。

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、基于数据库实现分布式锁</h3>

#### 1.1 悲观锁

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



**获取重入锁:**

```mysql
SELECT
  t0.id,
  t0.lock_name,
  t0.gmt_create 
FROM distribution_lock t0
WHERE t0.id = 1001 AND t0.node_number = 1;
```



**锁超时:**

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



#### 1.2 乐观锁



---

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、基于Redis实现分布式锁</h3>

##### 悲观锁



##### 乐观锁



---

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、基于Zookeeper实现分布式锁</h3>

##### 悲观锁



##### 乐观锁

