# MySQL逻辑架构
![MySQL逻辑架构图](https://static001.geekbang.org/resource/image/0d/d9/0d2070e8f84c4801adbfa03bda1f98d9.png)

## 连接器
连接器负责和客户端建立、维持和管理连接，以及权限验证等。

建立连接：
```bash
mysql -h host -u user -p
mysql -u xxxuser xxxdb
mysql -h100.64.1.152 -P6800 -uwm_trade_r -p5JPeuhS7IwJZHQ1S waimai_trade -A
```

最大连接数：
```sql
show variables like 'max_connections';
```

连接默认自动断开时间：
```sql
show variables like 'wait_timeout';
```

当由于连接数过多导致数据库不可用时，可以手动kill掉空闲连接。

### 长连接 vs 短连接
使用长连接可以减少建立连接的次数，但是过多长连接会消耗很多内存。

常用的长连接实现方式是使用数据库连接池。

## 查询缓存
建议不要使用查询缓存，尤其是对于更新频繁的数据库，查询缓存的命中率非常低。对于静态表，可以考虑使用查询缓存。

从MySQL 8.0开始，不再支持查询缓存。

## 分析器
词法分析、语法分析

## 优化器
选择索引、多表关联顺序等。

## 执行器
调用对应存储引擎的接口

## 存储引擎
负责数据的存储和提取。MySQL支持InnoDB、MyISAM、Memory等多种存储引擎。