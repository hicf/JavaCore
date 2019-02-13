<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">Redis持久化</h3>

![persistence](https://i.loli.net/2019/02/13/5c63bb6ce9d5f.jpeg)

Redis 是一种基于内存的数据库，一旦断电、重启，Redis 中的数据将不复存在。Redis 提供了 RDB 和 AOF 两个持久化的方式。当 Redis 实例重启时，可以使用已持久化的数据（文件）来还原内存数据集。



---

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">RDB</h3>

**RDB** 的全称是 *Redis DataBase* ，Redis 数据库。这种持久化方式是将当前Redis内存中的数据生成 **快照(Snapshot)** 文件保存到硬盘，简单粗暴。RDB也是Redis默认开启的持久化方式。



##### 触发持久化

![save](https://i.loli.net/2019/02/13/5c63c0174e045.png)

在之前已经废弃的 `save` 命令来实现RDB持久化数据，`save` 命令持久化时会一直阻塞，直到持久化完成。数据量越大，持久化过程带来的阻塞时间就越长。

![bgsave](https://i.loli.net/2019/02/13/5c63c02cc0753.png)

现在可以使用 `bgsave` 命令来代替 `save`。bg就是background，使用 `bgsave` 方式RDB持久化时，Redis工作进程会 fork 一个子进程，该子进程专门来负责耗时的RDB持久化，而工作进程会继续工作，阻塞时间只发生在短暂的 fork 阶段。



除了主动触发命令外，Redis还是提供了自动触发RDB持久化命令 `bgsave <seconds> <changes>` ，该命令表示在seconds秒内，Redis内存数据修改的次数达到changes次时，将触发RDB持久化。比如 `bgsave 7200 10` 表示在2个小时内，数据被更改10次时将自动触发RDB持久化。

（注意changes计数并不包含查询语句，当然被操作的key一定存在，否则也不会进行计数）

作者现在测试使用的是 `4.0.9` 版本，Redis在此版本默认开启的参数是 `save 900 1` 、`save 300 10` 、`save 60 10000` ，更加实际情况可自定义更改。



##### 数据快照存储

首先，工作目录默认为当前Redis目录，那么持久化的RDB数据快照将存储在该目录，可通过 `dir ./` 参数进行修改。生成的数据存储快照名称默认为 `dump.rdb` ，可通过 `dbfilename dump.rdb` 参数进行修改。RDB持久化先生存一个临时快照，待数据持久化完成，这个临时快照会替换原快照，以防持久化过程出现问题。

RDB存储快照默认是使用LZF算法压缩功能，压缩后的文件体积将大大减少。压缩文件但并不是十全十美，RDB快照压缩时也会消耗CPU，数据量越大，消耗CPU资源就越多，但官网还是极力推荐我们开始此功能。

RDB持久化时可能出现权限问题、存储空间不够用等问题，Redis 默认开启`stop-writes-on-bgsave-error yes`  参数，意味着出现 error时，Redis将停止向快照文件写入数据。

在Redis保存/恢复数据时，并不是一股脑进行保存/恢复，默认会对RDB快照文件进行检查，性能会受到影响(大约10%)，因此可以禁用它以获得最大的性能。(推荐开启)



---

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">AOF</h3>

**AOF** 的全称是 *Append Only File* ，仅附加文件，类似于MySQL的binlog。这种持久化方式是通过保存写命令来记录操作日志，在数据恢复时，重新执行已记录的写命令，来达到数据恢复。



##### AOF原理

![AOF](https://i.loli.net/2019/02/13/5c63dd9d9f120.png)

默认情况下，AOF是关闭的，可以通过修改 `appendonly no` 为 `appendonly yes` 来开启 AOF 持久化。内存的Output的带宽远大于磁盘Input的带宽，如果内存瞬时将大量的数据写入磁盘，必然发生IO堵塞，对于单线程工作的Redis也将会在性能上大打折扣。所以所有的写命令并不会直接写入AOF文件，而是先将写命令存储至AOF Buffer缓冲区，最后在同步至AOF文件。

生成的AOF文件名，默认为 `appendonly.aof` ，可以修改 `appendfilename "appendonly.aof"` 参数来更改生成的AOF文件名。



从缓冲区同步至磁盘，Redis提供了三种同步方式:

**always:**  写命令被追加到AOF缓存后，调用系统fsync函数将缓冲区的数据同步至磁盘，最后返回

**everysec:** 写命令被追加到AOF缓存后，调用系统write函数，在write函数执行完成，然后返回，会触发系统将对缓冲区的数据进行同步至磁盘

**no:** 写命令被追加到AOF缓存后，调用系统write函数，然后返回，后续的同步由系统负责

其中 `everysec` 不仅是默认的方式，也是推荐的方式。



##### AOF重写机制

AOF持久化文件存储的是Redis写命令，这比仅仅是数据的存储文件要大了很多，而且像如下的命令就可以考虑做很多优化:

```shell
127.0.0.1:6379> set user:10001 Kevin
OK
127.0.0.1:6379> del user:10001
(integer) 1
127.0.0.1:6379> set user:10002 Jack
OK
127.0.0.1:6379> del user:10002
(integer) 1
127.0.0.1:6379> set user:10003 Tony
OK
127.0.0.1:6379> del user:10003
(integer) 1
```

上述命令都是些命令，最终是空数据，如果都存储到AOF文件，不仅增大文件体积，而且在恢复数据时会有很多不必要的执行命令。AOF重写机制对持久化的命令做了优化。

在AOF重写机制中，首先遍历数据库，如果数据库为空库，则跳过该库。对于非空的数据库，则遍历所有的key，如果key过期则跳过该key，如果key有过期时间则设置key的过期时间。对于不同类型的数据，用不同的、最简单的写命令来保存，比如下面的命令:

```shell
# 假设一开始 user 对应的 value 为空
lpush user kevin

lpush user jack

lpush user tony

rpop user

lpop user
```

可以重写成:

```shell
lpush user jack
```

这样大大减少了不必要的命令，在存储AOF文件时，文件的体积也会大大减少，而且恢复数据的效率也会大大提升。



AOF的重写机制并不是随机触发的，默认是当前写入AOF文件大小较上次重写后文件大小增长了 `100%` 时就触发重写机制。比如上次重写后的文件大小是1G，经过一段时间，AOF文件体积增长了 `1G` ，那么增长率就是 `100%` ，就会触发AOF重写。可以通过设置 `auto-aof-rewrite-percentage 100` 来调节文件增长率的触发阈值。

如果AOF的文件较少，不足以影响太大性能，所以没必要重写AOF文件。可以通过 `auto-aof-rewrite-min-size 64mb` 来设置AOF文件重写的最小阈值。



`aof-load-truncated yes` 参数表示，在Redis恢复数据时，最后一条命令可能不完整，开启则表示忽略最后一条不完整的命令。



---

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">RDB和AOF对比</h3>

RDB开始压缩后，生成一个体量更小、更紧凑的二进制文件，占有空间小。RDB不仅有着较快的数据恢复能力，而且可以直接复制RDB文件至其他Redis实例进行回复数据，有着良好的容灾体验。RDB适用于定时复制、全量复制、快速恢复的场景，由于RDB文件生成时间长，数据有存储延迟，不适用于近实时持久化的场景。



AOF以追加命令的方式，持续的、近实时的写入文件（秒级别）。而且，在一定程度时，AOF不断的重写持久化文件，不断的优化。特别是在数据完整性上要比RDB好很多，例如，使用默认的数据同步策略，Redis在发生服务器断电等重大事件时，可能只丢失一秒钟的写入时间，或者在Redis进程本身发生错误但操作系统仍在正常运行时丢失一次写入时间。所以AOF比较适合对数据实时性和数据完整性要求比较高的场景。AOF相较于RDB，缺点是文件相对较大，而且恢复数据时因为要执行全部的写命令，所以数据恢复比较慢，特别在是在被依赖的应用系统高负载的情况下，较长时间的数据恢复，对后端系统是不可容忍的。



但AOF和RDB持久性可以同时启用，不会出现问题。如果在启动时启用了AOF，redis将加载AOF，即具有更好的持久性保证的文件。而且在较新的版本还支持混合模式。



#### RDB-AOF混合持久化

Redis从 `4.0` 开始支持RDB-AOF混合持久化，默认是关闭状态，可以通过 `aof-use-rdb-preamble yes` 开启。AOF文件在重写之后，将生成一份记录已有数据的RDB快照文件，再生成一份记录最近写命令的AOF文件，作为对RDB快照的补充。这样的混合持久化模式，将兼具RDB和AOF的双优特性。