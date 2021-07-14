# ACID
- atomicity, a transaction should be atomic; that is, it should either be completely executed or not executed
- consistency, a transaction should leave the database in a consistent state
- isolation, uncompleted transactions should not be visible to other users of the database; that is, until transactions are complete, they should remain isolated
- durability, once written to the database, a transaction should be permanent or durable

# 隔离级别（SQL标准）
- READ UNCOMMITTED，一个事务还未提交时，它做的变更就能被其他事务看到
- READ COMMITTED，一个事务提交后，它做的变更才能被其他事务看到
- REPEATABLE READ，一个事务执行过程中看到的数据，总是和这个事务在启动时看到的数据是一致的
- SERIALIZABLE，串行化，读写都加锁

MySQL在实现上会创建一个视图，访问的时候以视图的逻辑结果为准。在“读提交”隔离级别下，这个视图是在每个SQL语句开始执行的时候创建的。在“可重复读”隔离级别下，这个视图是在事务启动时创建的。“读未提交”隔离级别下直接返回记录上的最新值，没有视图概念。“串行化”隔离级别下直接用加锁的方式来避免并行访问。

当前隔离级别：
```sql
show variables like 'transaction_isolation';
```

# 长事务
事务是否自动提交：
```sql
show variables like 'autocommit';
```

单个语句最长执行时间：
```sql
show variables like 'max_execution_time';
```

查询长事务：
```sql

select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60
```

# MVCC
InnoDB里面每个事务有一个唯一的事务id，称为transaction id. 它是在事务开始的时候向InnoDB的事务系统申请的，按申请顺序严格递增。

每行数据有多个版本，每次事务更新数据的时候，会生成一个新的版本，并且把事务id和这个版本关联，记为row trx_id.

![image](https://static001.geekbang.org/resource/image/68/ed/68d08d277a6f7926a41cc5541d3dfced.png)

图中的三个虚线箭头就是undo log, V1、V2、V3并不是物理上真实存在的，而是每次需要的时候根据当前版本和undo log计算出来的。

事务启动时，InnoDB会为事务构造一个数组，这个数组里是当前活跃事务id. 数组里面事务id的最小值记为低水位，当前系统已经创建过的事务id的最大值加1记为高水位，这个视图数组和高水位，就组成了当前事务的一致性视图（consistent read view）。而数据版本的可见行，就是基于数据的row trx_id和这个一致性视图的对比结果得到的。

InnoDB利用了“所有数据都有多个版本”这个特性，实现了秒级创建快照的能力。

更新数据都是先读后写，而这个读，只能读当前的值，称为当前读（current read）。

除了update语句外，select语句如果加锁，也是当前读。
```sql
select k from t where id=1 lock in share mode;

select k from t where id=1 for update;
```

可重复读的核心就是一致性读（consistent read）。事务更新数据时，只能用当前读。如果当前的记录的行锁被其他事务占用，就需要进入锁等待。