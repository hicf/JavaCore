> 目前阿里云推出的 *云数据库RDS* 的版本有 `5.7`、`5.6`、`5.5` 三个版本，以下测试是基于 `5.7`版本。全文也会随着版本流行度进行及时更新，原文更新地址：https://github.com/about-cloud/JavaCore

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">优化必备的 EXPLAIN 命令</h3>

> `EXPLAIN`  是用来查询 SQL 的执行计划，

用法：

```mysql
EXPLAIN SELECT [字段...] FROM TABLE;
```

结果：

```mysql
+----+-------------+-------+------------+-------+---------------+--------------+---------+------+------+----------+-----------------------+
| id | select_type | table | partitions | type  | possible_keys | key          | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+-------+------------+-------+---------------+--------------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | t0    | NULL       | range | idx_trade_id  | idx_trade_id | 8       | NULL |    2 |   100.00 | Using index condition |
+----+-------------+-------+------------+-------+---------------+--------------+---------+------+------+----------+-----------------------+
1 row in set (0.02 sec)
```





<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">索引优化</h3>

#### 1、禁止无边界范围查询 `!=` 、`<`  、`>` 、`=<` 、`>=` ，否则不会命中索引

*反例*：:x:

```mysql
SELECT t0.product_id
FROM product t0
WHERE t0.stock_quantity >= '5';
```

:pushpin:**应指定上下边界**

正例：:white_check_mark:

```mysql
SELECT t0.product_id
FROM product t0
WHERE t0.stock_quantity >= '5' AND t0.stock_quantity <= '999999';
```



#### 2、禁止无边界范围查询 `NOT IN`  ，否则不会命中索引

*反例*：❌

```mysql
SELECT t0.product_id
FROM product t0
WHERE t0.product_status NOT IN ('已下架', '已删除', '无库存');
```

📌**应指定范围**

正例：✅

```mysql
SELECT t0.product_id
FROM product t0
WHERE t0.product_status IN ('正常销售', '促销中', '可预订');
```

*使用 `IN` 时也控制范围大小，不推荐超过 `1000`，否则影响性能，得不偿失。



#### 2、禁止 ~~`左模糊`~~ 或 ~~`全模糊`~~ 查询，否则不会命中索引

```mysql
SELECT t0.product_id
FROM product t0
WHERE t0.product_name LIKE ('');
```





严禁 SELECT 使用 `*` 代替查询字段，应一一指明字段。



##### ~~NOT IN ()`~~ 不会命中索引：:x:

```sql
SELECT
  t0.product_id
FROM product t0
WHERE t0.product_type NOT IN ('日用品', '电子产品');
```
应改为：:white_check_mark:
```sql
SELECT
  t0.user_name
FROM product t0
WHERE t0.product_type NOT IN ('日用品', '电子产品');
```

##### 2、禁止 ~~`左模糊`~~ 或 ~~`全模糊查询`~~ ，否则不会命中索引：
```sql
SELECT t0.user_name
FROM user to
WHERE t0.user_name LIKE '%zhangsan%';
```
应改为（或者使用`搜索引擎`）：
```sql
SELECT t0.user_name
FROM user to
WHERE t0.user_name LIKE 'zhangsan%';
```

##### 3、字段的默认值不要为 ~~`null`~~，否则不会命中索引：
> 填充默认值

##### 4、判断是否为空，只能使用 `ISNULL()`函数（否则结果有出入）：(待验证)
```sql
SELECT ISNULL(t0.user_name)
FROM user to
```

##### 5、在 ~~`字段上计算`~~ 后，不会命中索引：
```sql
SELECT t0.user_name
FROM user to WHERE FROM_UNIXTIME(t0.gmt_create) < CURDATE();
```
应改为：
```sql
SELECT t0.user_name
FROM user to WHERE t0.gmt_create < FROM_UNIXTIME(CURDATE());
```

##### 6、`varchar`查询性能比 ~~`bigint`~~ 好：
> 因为 在 ~~`bigint`~~ 类型字段上会`全表扫描`，而在 `varchar` 上每个字符判断会走`索引`，这样尽量避免全表扫描。

##### 7、小数类型使用 `decimal`，禁止使用 ~~`float`~~ 与 ~~`double`~~： 
> ~~`float`~~ 与 ~~`double`~~存储数据时，可能会损失精度，进而判断的时候导致结果不准，`强制` 使用 `decimal` 数据类型。

##### 8、表达`是否`的概念时，字段使用`is_`开头，数据类型使用 `unsigned tinyint`类型，`1`表示`是`，`0`表示`否`。

##### 9、`任何非负数`都必须声明为`unsigned`类型：
> 比如年龄、状态码等等

##### 10、如果存储的`字符串长度几乎相等`，`必须`使用`char`定长字符串类型：
> 比如 11 位长度的手机号。

##### 11、有时候是不需要建索引：
> 性别字段、状态，这种不同值很少的字段是不需要建索引的。

##### 12、单表行数超过 `500万行` 或者单表容量超过 `2G`，才推荐分表；

##### 13、进行 `UPDATE` 或 `DELETE` 时，必先 `SELETE`，避免出现`误删`数据；

##### 14、`组合索引` 的 `最左前缀原则`：
> 建立 `(id_sex_city)组合索引`会自动建立 `id`、`id_sex`和`id_sex_ciry`三个索引。所以下面三个语句都会 `命中索引`：
```sql
-- statement1：
SELECT t0.user_name
FROM user t0
WHERE t0.id = '1001' AND t0.sex = '1' AND t0.ciry = 'Shanghai';

-- statement2：
SELECT t0.user_name
FROM user t0
WHERE t0.id = '1001' AND t0.sex = '1';

-- statement3：
SELECT t0.user_name
FROM user t0
WHERE t0.id = '1001';
```
> 但下列语句 ~~`不会命中索引`~~，因为根据`组合索引`的`最左前缀原则`只会生成靠左、逐渐递进的索引：
```sql
-- statement4：
SELECT t0.user_name
FROM user t0
WHERE t0.sex = '1' AND t0.ciry = 'Shanghai';

-- statement5：
SELECT t0.user_name
FROM user t0
WHERE t0.id = '1001' AND t0.ciry = 'Shanghai';

-- statement6：
SELECT t0.user_name
FROM user t0
WHERE t0.ciry = 'Shanghai';

-- 略...
```

> （组合索引、复合索引、联合索引都是一个意思）。

##### 15、尽量使用不要让数据库帮我们做数据类型转换：
```sql
SELECT t0.user_name FROM user t0 WHERE t0.user_id = 1000000378;
```
上述语句会扫描全表，应改为下面代码：

```sql
SELECT t0.user_name FROM user t0 WHERE t0.user_id = '1000000378';
```

##### 16、对于多表 `JOIN` 时的 `ON` 条件中`字段类型`一定要一致，否则也不会命中索引.