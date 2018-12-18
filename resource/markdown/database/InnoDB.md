<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">MySQL存储引擎</h3>



存储引擎对比

| 引擎    | 存储限制 | 支持事务 | 全文索引 | 哈希索引 | 数据缓存 | 支持外键 |
| ------- | -------- | :------: | :------: | :------: | :------: | :------: |
| InnoDB  | 64TB     |    ✅     |   :x:    |    ✅     |    ✅     |    ✅     |
| MyISAM  | 256TB    |   :x:    |    ✅     |    ✅     |   :x:    |   :x:    |
| Memory  | RAM      |   :x:    |   :x:    |    ✅     |   N/A    |   :x:    |
| Archive | None     |   :x:    |   :x:    |   :x:    |   :x:    |   :x:    |

