<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">Redis持久化</h3>

Redis 是一种基于内存的数据库，一旦断电、重启，Redis 中的数据将不复存在。Redis 提供了 RDB 和 AOF 两个持久化的方式。当 Redis 实例重启时，可以使用已持久化的数据（文件）来还原内存数据集。



---

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">RDB</h3>

**RDB** 的全称是 *Redis DataBase* ，Redis 数据库。这种持久化方式是将当前Redis内存中的数据生成快照文件保存到硬盘，简单粗暴。



在之前的、已经废弃的 `save` 命令来实现RDB持久化数据，因为Redis是单线程工作的，`save` 命令持久化时会一直阻塞，直到持久化完成。数据量越大，持久化带来的阻塞时间就越长。

现在可以使用 `bgsave` 命令来代替 `save`。bg就是background，使用 `bgsave` 方式RDB持久化时，Redis工作进程会 fork 一个子进程，该子进程专门来负责RDB持久化，而工作进程会继续工作，阻塞时间只发生。





---

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">AOF</h3>