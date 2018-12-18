Linux平台MySQL的有关目录：

| 目录               | 描述                       |
| ------------------ | -------------------------- |
| /usr/bin           | 客户端和脚本               |
| /usr/sbin          | MYSQLD服务器               |
| /usr/lib/mysql     | 库                         |
| /usr/share/info    | 信息格式的手册             |
| /usr/share/man     | UNIX帮助页                 |
| /usr/share/mysql   | 错误消息、字符集、示例配置 |
| /usr/include/mysql | 头文件                     |
| **/var/lib/mysql** | 日志文件和数据库           |



```shell
# 关于MySQL启动与关闭服务的命令分别表示：启动、停止、重启、查看状态
$ service mysqld start | stop | restart | status 
```

查看数据库的引擎情况：

```mysql
SHOW ENGINES;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
9 rows in set (0.00 sec)
```

