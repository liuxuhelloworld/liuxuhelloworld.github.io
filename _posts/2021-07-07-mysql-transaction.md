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

查询长事务：
```sql

select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60
```