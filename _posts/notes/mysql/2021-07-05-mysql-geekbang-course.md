# MySQL逻辑架构
![MySQL逻辑架构图](https://static001.geekbang.org/resource/image/0d/d9/0d2070e8f84c4801adbfa03bda1f98d9.png)

## 连接器
连接器负责和客户端建立、维持和管理连接，以及权限验证等。

建立连接：
```bash
mysql -h host -u user -p
mysql -u xxxuser xxxdb
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

# Write-Ahead Logging
简单说就是先写日志，再写磁盘，将写磁盘从随机写转换成了顺序写，这样可以提高更新效率。

# redo log
InnoDB特有的日志，在更新语句时，先写redo log，然后更新内存，这样就算更新完成了。后面InnoDB会在合适的时候再将操作记录更新到磁盘。

```sql
show variables like 'innodb_flush_log_at_trx_commit'; 
```
这个参数设置用于控制redo log的写入策略：
- 设置为0，表示每次事务提交都只写到redo log buffer
- 设置为1，表示每次事务提交都持久化到磁盘
- 设置为2，表示每次事务提交都只写到page cache

redo log是物理日志，记录的是“在某个数据页上做了什么修改”。

redo log是固定大小的，通过wirte pos和check point循环读写。对于TB级别的磁盘，建议将redo log设置为4个文件，每个文件1GB大小。
```sql
show variables like 'innodb_log_file_size';
show variables like 'innodb_log_files_in_group';
```

![redo log示意图](https://static001.geekbang.org/resource/image/16/a7/16a7950217b3f0f4ed02db5db59562a7.png)

有了redo log，InnoDB就可以保证即使数据库发生异常重启，之前提交的记录也不会丢失，这种能力称为crash-safe.

# binlog(归档日志)
MySQL server层自己的日志。

binlog是逻辑日志，记录的是语句的原始逻辑，比如“给ID=2的这一行的c字段加1”。binlog是可以“追加写”的，写到一定大小后会切换到下一个。

binlog格式：
- statement，记录的是SQL语句原文，可能会出现主备不一致的unsafe情形，比如在主备环境下使用了不同的索引
- row，记录的是实际处理的行信息，不会有主备不一致，缺点是占用空间较大
- mixed，折中方案，MySQL判断是否有可能导致主备不一致，如果有可能，使用row格式，否则使用statement格式

越来越多的场景依赖于把binlog格式设置为row，这样的好处是方便做数据恢复。

数据恢复或新增备库等操作就会用到binlog，一般采用的方法是基于全量备份重放binlog.

binlog写入逻辑：事务执行过程中，先把日志写到binlog cache，事务提交的时候，再把binlog cache写到binlog文件中。

```sql
show variables like 'binlog_cache_size';
```
这个参数用于控制单个线程内binlog cache所占内存的大小。

![binlog写盘状态](https://static001.geekbang.org/resource/image/9e/3e/9ed86644d5f39efb0efec595abb92e3e.png)

```sql
show variables like 'sync_binlog';
```
write和fsync的时机由*sync_binlog*参数控制。*sync_binlog*为0表示每次提交事务都只write，不fsync；*sync_binlog*为1表示每次提交事务都write并且fsync；*sync_binlog*为N表示每次提交事务都write，累积N个事务后fsync.

一般的，将*sync_binlog*和*innodb_flush_log_at_trx_commit*均设置为1是比较保险的做法。当出现IO瓶颈时，可以考虑将*sync_binlog*设置为为100到1000中的某个值，风险是如果机器异常重启，会丢失最近的N个事务的binlog日志。也可以考虑将*innodb_flush_log_at_trx_commit*设置为2，风险是如果机器掉电会丢失数据。

# update执行流程

![update执行流程](https://static001.geekbang.org/resource/image/2e/be/2e5bff4910ec189fe1ee6e2ecc7b4bbe.png)

redo log的写入分两个阶段，第一个阶段是prepare状态，写入binlog后，在提交事务时将redo log置为commit状态，这就是两阶段提交。

为什么需要两阶段提交？因为只有这样才能保证在发生异常时，server层和InnoDB的状态是一致的：
- 如果在redo log是prepare状态而写binlog之前发生了crash，由于此时binlog还没写，redo log也未commit，崩溃恢复时这个事务会回滚。binlog没写，也不会传到备库
- 如果在binlog写完而redo log还未commit之前发生了crash，那么崩溃恢复时会判断对应的binlog是否存在并且完整，如果是，则提交事务，否则回滚事务

# undo log
在MySQL中，每条记录在更新的时候都会同时记录一条回滚操作。记录上的最新值，通过回滚操作，可以得到前一个状态的值。

![回滚日志示意图](https://static001.geekbang.org/resource/image/d9/ee/d9c313809e5ac148fc39feff532f0fee.png)

不同时刻启动的事务会有不同的read-view，当没有比某个回滚日志更早的read-view时，回滚日志会被删除。

# 为什么MySQL会“抖”一下？
你可能碰到过这样的场景，一条SQL语句，正常执行的时候特别快，但是有时不知道怎么回事，会变得特别慢，也很难复现，就像是数据库“抖”了一下。

一个可能的原因是刷脏页（flush）导致的，可能是redo log满了或者内存中脏页过多。

```sql
show variables like 'innodb_io_capacity';
```
这个参数是告诉InnoDB你的磁盘能力，应当参考磁盘的IOPS来设置，如果设置的偏低了，就限制了flush的速度，就会造成脏页累计，影响查询和更新性能。

InnoDB的flush速度会参考两个因素：一个是脏页比例，一个是redo log写盘速度。

```sql
show variables like 'innodb_max_dirty_pages_pct';
```
这个参数设置的是脏页比例上限。

为了避免这种抖动，一方面要合理地设置*innodb_io_capacity*，另一方面要多关注脏页比例，尽量控制在75%以内。

# 误删数据
如果使用DELETE误删了数据，可以通过修改binlog后重放恢复原数据。当然，需要**binlog_format=row**和**binlog_row_image=FULL**.

如果误删了表或库，那只能通过全量备份+增量日志的方式恢复数。当然，需要有定期的全量备份并且实时备份binlog.

MySQL 5.6引入了延迟复制备库，可以设定某个备库和主库保持N秒的延迟，这样如果发现及时可以降低数据被误删的影响范围。

rm命令误删数据，如果只是集群的某个节点，影响是可控的，HA系统会选出一个新的主库，保证整个集群的正常工作。如何避免整个集群被删除呢？跨机房备份一定程度上可以解决这个问题。

# 全局锁
全局锁是对整个数据库实例加锁，典型应用场景是做全库逻辑备份。

备份为什么要加锁？因为如果不加锁的话，备份得到的库不是一个逻辑时间点，备份的视图是逻辑不一致的。

```sql
flush tables with read lock;
```
这个命令就让整个库处于只读状态。

mysqldump配合--single-transaction，通过可重复读也可以用来备份，前提是存储引擎要支持事务和对应的隔离级别。

```sql
set global readonly=true
```
也可以达到全库只读的状态，但是不建议使用。

# 表级锁
## 表锁
```sql
lock tables ... read;
lock tables ... write;
```
表锁一般是在数据库引擎不支持行锁的时候才会被用到。

## 元数据锁（meta data lock, MDL）
MDL不需要你显式使用，对表做增删改查时会自动加MDL读锁，对表结构做变更时会自动加MDL写锁。

MDL读锁之间不互斥，因此可以有多个线程同时对一张表增删改查。

MDL读写锁之间、写锁之间是互斥的，用来保证变更表结构操作的安全性，因此如果有两个线程要同时给一个表加字段，其中一个要等另一个执行完才能开始执行。

# 行锁
行锁是存储引擎层实现的，不是所有存储引擎都支持行锁，比如MyISAM就不支持行锁，不支持行锁就得使用表锁，同一张表任何时刻只能有一个更新在执行，这就大大限制了业务并发度。

InnoDB的行锁在需要的时候自动加，但并不是不需要了就立刻释放，而是在事务提交时释放，这就是两阶段锁协议。因此如果你的事务中需要锁多个行，要把最可能造成锁冲突、最可能影响并发度的更新往后放。

# 间隙锁
为了解决幻读，InnoDB引入了间隙锁(Gap Lock)，间隙锁锁的是两个值之间的空隙，也就是说，在扫描行的过程中，不仅加了行锁，还给行两边的空隙加上了间隙锁。间隙锁和行锁合称next-key lock，每个next-key lock是前开后闭区间。

注意，间隙锁是在可重复读隔离级别下才会生效的。如果使用读提交，就没有间隙锁了，但要注意可能出现的数据和日志不一致问题，需要把binlog格式设置为row.

简单了解的加锁规则：
- 加锁的基本单位是next-key lock
- 查询过程中访问到的对象才加锁
- 索引上的等值查询，向右遍历时且最后一个值不满足等值条件的时候，next-key lock退化为间隙锁
- 索引上的等值查询，如果是唯一索引，next-key lock退化为行锁

注意，这里等值查询指的是通过树搜索的方式定位记录。

# 死锁和死锁检测
当并发系统中不同线程出现循环资源依赖，涉及的线程都在等待别的线程释放资源时，就会导致这几个线程都进入无限等待的状态，即死锁。

出现死锁以后，一种策略是等待，直到超时，默认超时时间是50秒。
```sql
show variables like 'innodb_lock_wait_timeout';
```
这个超时时间不能设置的太小，因为可能不是死锁，而只是锁等待，超时时间设置的太小的话会有很多误伤。

另一种策略是开启死锁检测，检测到死锁后回滚某一个事务，让其他事务得以继续执行。死锁检测默认是开启的。
```sql
show variables like 'innodb_deadlock_detect';
```
死锁检测的问题是可能占用大量CPU资源，尤其是热点行的高并发更新时。表现出来的现象就是CPU利用率飙高，但每秒执行不了几个事务，因为CPU资源都消耗在死锁检测上了。

那么如何解决热点行更新问题呢？最根本的办法是控制好并发度，在数据库中间件或数据库源码里增加限流控制。如果没有这样技术手段，可以将针对一行的逻辑改成多行。

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

# 幻读
幻读指的是一个事务在前后两次查询同一个范围的时候，后一次查询看到了前一次查询没有看到的行。注意，幻读专指”新插入的行“。

在可重复读隔离级别下，普通的查询是快照读，不会出现幻读，幻读在当前读下才会出现。

幻读有什么问题呢？幻读破坏了可重复读的语义，还会导致数据与日志不一致的问题。

![image](https://static001.geekbang.org/resource/image/5b/8b/5bc506e5884d21844126d26bbe6fa68b.png)

上图的逻辑基于假设Q1时只d=5的这一行加了锁，C的情形就是幻读，B的情形不算幻读，幻读专指“新插入的行”。

如果我们把所有扫描到的行都加上写锁，那么就是下图：
![image](https://static001.geekbang.org/resource/image/34/47/34ad6478281709da833856084a1e3447.png)

B的情形没有问题了，但是C的情形依然是幻读，也就是说，即便已经把所有记录的加上了锁，还是没解决id=1这一行的插入和更新。

# 索引数据模型
提高查询效率的数据结构有很多，常见的有：
- 哈希表：哈希表适用于只有等值查询的场景，区间查询很慢
- 有序数组：有序数组做等值查询和区间查询都很快，但是插入记录的成本很高，适用于静态存储
- 搜索树：使用N叉树而不是简单地使用二叉树是为了减少读写磁盘的次数。以InnoDB的一个整数字段索引为例，这个N差不多是1200。这棵树高是4的时候，就可以存1200的3次方个值，这已经17亿了。考虑到树根的数据块总是在内存中的，一个 10亿行的表上一个整数字段的索引，查找一个值最多只需要访问3次磁盘。其实，树的第二层也有很大概率在内存中，那么访问磁盘的平均次数就更少了

# InnoDB索引
MySQL索引是在存储引擎层实现的。

InnoDB的表都是根据主键以索引的形式存放的，这种存储方式称为索引组织表。

InnoDB使用B+树维护索引，每一个索引在InnoDB中对应一颗B+树。

根据叶子节点的内容，可以分为：
- 主键索引：主键索引叶子节点存储的是整行数据，主键索引也称为聚簇索引，根据主键索引查询只需要搜索一颗B+树
- 非主键索引：非主键索引叶子节点存储的是主键值，非主键索引也称为二级索引，根据非主键索引查询需要搜索两颗B+树，即需要回表

由于主键索引树的叶子结点是数据，普通索引树的叶子结点是主键值，所以普通索引树比主键索引树小很多。

插入数据时需要做一些操作来维护B+树，可能涉及页分裂和页合并。

什么情况下需要重建索引？索引可能因为删除，或者页分裂等原因，导致数据页有空洞，重建索引的过程会把数据按顺序插入，这样页面的利用率更高，也就是索引更紧凑、更省空间。

## 自增主键
某些建表规范要求使用自增主键，为什么？
- 使用自增主键的好处是每次都是追加操作，不会触发叶子节点的分裂，如果使用业务字段做主键，不容易保证有序插入，写数据的成本就会大一些
- 非主键索引的叶子节点存储的是主键值，使用自增主键占用的空间小
- 因此，从时间和空间两方面的效率来看，使用自增主键一般都是更好的选择

有没有场景适合用业务字段做主键，而不使用自增主键呢？有的，如果你的业务场景只有一个索引，并且是唯一索引，那么也可以使用业务字段做主键。

为什么自增主键会不连续？主要原因在于获取自增主键id时加的锁不是一个事务锁，每次申请完就马上释放，为了保证性能，InnoDB在插入失败或回滚时不会回退自增主键id。另外，批量插入语句会采用批量申请自增id的策略，可能有部分id被浪费。总之，自增主键保证递增，但不保证连续。

# 覆盖索引
如果根据某个索引查询时不需要回表，这个索引就称为覆盖索引，覆盖索引是一种常用的优化手段。

比如，公民信息表，以身份证号做了索引，还有必要以身份证号+名字做联合索引吗？如果根据身份证号查询名字是高频的，那么以身份证号+名字做联合索引是合适的，这样就用到了覆盖索引，而无需回表。

# 最左前缀原则
B+树结构的索引，可以利用索引的“最左前缀”来定位记录。只要满足最左前缀，就可以用到索引来加速查询，这个最左前缀可以是联合索引的最左N个字段，也可以是字符串索引的最左N个字符。

在建立联合索引的时候，如何安排索引内的字段顺序呢？
- 一个原则是，如果通过调整顺序，可以少维护一个索引，那么这个顺序往往是需要优先考虑采用的。比如，已经有了(a,b)的联合索引，一般就不需要单独在a上建立索引了
- 那么如果需要单独查b呢？查询条件里只有b的语句，是无法使用(a,b)这个联合索引的，此时就不得不为b单独再建一个索引
- 如果既有a和b的联合查询，又有分别查询a或b，那么应当考虑将占用空间更小的建立单独索引

## 前缀索引
```sql
ALTER TABLE user ADD INDEX index1(email);

ALTER TABLE user ADD INDEX index2(email(6));
```
即我们可以定义字符串的一部分作为索引，这种称为前缀索引。

前缀索引的好处是节省空间，代价是可能增加额外的记录扫描次数，降低了查询效率。关键点在于前缀长度，选择合适的长度，既可以节省空间，又不会额外增加太多查询成本。

前缀索引还有一个不利影响是使用了前缀索引就用不上覆盖索引对查询性能的优化了，因为前缀索引总是要回表的。

如果要建立索引的字段的前n个字符区分度很差怎么办？使用它们建立前缀索引就不合适了。如果末尾的子串具备合理的区分度，那么可以考虑倒序存储+前缀索引。

除了前缀索引，还有没有其他方式优化字符串字段的索引问题呢？有的，可以考虑使用hash字段，即对字符串字段做hash运算，记录hash值，然后在hash值上建立索引。

倒序存储+前缀索引、hash字段这两种优化方式最大的限制就是不支持范围查询，只能做等值查询。

# 索引下推
MySQL 5.6引入，可以在索引遍历过程中，对索引中包含的字段先做判断，不满足查询条件的直接过滤，减少回表次数。

# 唯一索引 vs 普通索引
唯一索引和普通索引在读的效率上差别微乎其微，因为InnoDB的数据是按数据页为单位来读写的，当需要读一条记录的时候，并不是将这个记录从磁盘读出来，而是以页为单位，整体读入内存。

唯一索引和普通索引在写的效率上差别如何？这当中涉及到change buffer这个机制。

当需要更新一个数据页时，如果数据页在内存中就直接更新，而如果这个数据页还没有在内存中的话，在不影响数据一致性的前提下，InnoDB会将这些更新操作缓存在change buffer中，这样就不需要从磁盘中读入这个数据页了。在下次查询需要访问这个数据页的时候，将数据页读入内存，然后执行change buffer中与这个页有关的操作。通过这种方式就能保证这个数据逻辑的正确性。将change buffer中的操作应用到原数据页，得到最新结果的过程称为merge。除了访问这个数据页会触发merge外，系统有后台线程会定期merge。在数据库正常关闭（shutdown）的过程中，也会执行merge操作。显然，如果能够将更新操作先记录在change buffer，减少读磁盘，语句的执行速度会得到明显的提升。

唯一索引不会使用change buffer，因为需要判断是否违反唯一性原则，需要将数据页读入内存，将数据从磁盘读入内存涉及随机IO的访问，是数据库里面成本最高的操作之一。已经读入到内存了，直接更新内存就可以了。

普通索引可以使用change buffer，change buffer减少了随机磁盘访问，所以对更新性能的提升是会很明显的。

那么是所有普通索引的场景，都适合使用change buffer吗？对于写多读少的业务来说，change buffer可以大大提高效率，但是如果写完以后马上就读，change buffer反而可能会起反作用。

那么唯一索引和普通索引到底怎么选呢？一般的，建议尽量使用普通索引，如果写操作后马上要读，可以关闭change buffer.

```sql
show variables like 'innodb_change_buffering';
```

## change buffer vs redo log
通过下面这张图来理解change buffer和redo log的关系：

![change buffer vs redo log](https://static001.geekbang.org/resource/image/98/a3/980a2b786f0ea7adabef2e64fb4c4ca3.png)

change buffer主要节省的是随机读磁盘的IO消耗，redo log主要节省的是随机写磁盘的IO消耗。

# 为什么MySQL可能选错索引？
选择索引是优化器的工作，优化器会依据扫描行数、是否使用临时表、是否排序等多种因素决定使用哪一个索引。

扫描行数是怎么计算的呢？MySQL并不能精确计算扫描行数，只能根据统计信息来预估扫描行数。这个统计信息就是索引的“区分度”。显然，一个索引上不同的值越多，这个索引的区分度就越好。而一个索引上不同的值的个数，我们称之为“基数”（cardinality）。也就是说，这个基数越大，索引的区分度越好。可以通过如下语句查看索引的信息：
```sql
show index from xxxtable;
```

这个基数是怎么得到的？通过采样统计，采样统计的时候，InnoDB默认会选择N个数据页，统计这些页面上的不同值，得到一个平均值，然后乘以这个索引的页面数，就得到了这个索引的基数。而数据表是会持续更新的，索引统计信息也不会固定不变。所以，当变更的数据行数超过1/M的时候，会自动触发重新做一次索引统计。由于是采样统计，这个基数是很容易不准的。如果你发现explain的结果和实际差别较大，可以通过如下命令来重新统计索引信息：
```sql
analyze table xxxtable;
```

MySQL选择了错误的索引时怎么办？
- 如果是索引统计信息不准，可以通过analyze table t 命令，重新统计索引信息
- 使用force index强制使用某个索引，这种方法不灵活
- 修改查询语句，引导MySQL使用合适的索引
- 新增更合适的索引或者删除误用的索引

# 为什么看起来使用了索引但是效率很差？
```sql
SELECT COUNT(*) FROM xxx WHERE MONTH(created_at) = 7;
```
对索引字段做函数操作，可能会破坏索引值的有序性，所以就不能使用树搜索功能来快速定位，而不得不使用全索引扫描来遍历索引树。

```sql
SELECT * FROM xxx WHERE xxxstr = 123456;
```
在MySQL中，字符串和整数做比较，是将字符串转换成整数。也就是说，上面的语句等效于：
```sql
SELECT * FROM xxx WHERE CAST(xxxstr AS SIGNED INT) = 123456;
```
所以这也是对索引字段做函数操作，效率差是因为没有用到树搜索功能。

还有一种看起来使用了索引但是效率很差的例子是隐式字符编码转换，比如uft8转换utf8mb4，也等效于对索引字段做函数操作，导致用不到树搜索功能。

# DDL
## ALTER
```sql
ALTER TABLE xxx ENGINE=InnoDB;
```
这个语句用来重建表。

为什么需要重建表呢？因为经过大量增删改的表，数据页可能存在很多hole，重建表可以去掉这些hole，从而达到收缩表空间的目的。

重建表的过程中会先获取MDL写锁，但是这个写锁在真正拷贝数据之前就退化成读锁了，读锁不会阻塞增删改操作，这样重建表就可以看作是Online的。为什么不直接释放锁呢？为了防止其他线程对这个表同时做DDL.

重建表需要扫描原表数据和构建临时文件，对于很大的表来说，这个操作是很消耗IO和CPU的，所以操作要谨慎。

# DML
## COUNT
```sql
SELECT COUNT(*) FROM xxx;
```
在不同的引擎中，COUNT(\*)有不同的实现方式：
- MyISAM引擎把一个表的总行数存在了磁盘上， 执行COUNT(\*)时直接返回
- InnoDB引擎需要把数据一行一行的从引擎里面读出来，累计计数

为什么InnoDB需要一行一行计数呢？因为由于多版本并发控制（MVCC），对于同一时刻的多个count(\*)查询，应当返回的结果可能是不同的。

按效率粗略排序的话，count(字段) < count(主键) < count(1) or count(*)

## ORDER BY
```sql
SELECT xxx FROM xxx WHERE xxx ORDER BY xxx
```
ORDER BY可能是在内存中完成，也可能需要使用外部排序，这取决于排序所需的内存和sort_buffer的大小。
```sql
show variables like 'sort_buffer_size';
```
如果要排序的数据量小于sort_buffer_size，排序就在内存中完成。如果排序数据量太大，就不得不利用磁盘临时文件辅助排序。sort_buffer_size越小或者要返回的字段越多，sort_buffer里能同时放下的行数就越小，需要的临时文件数就越多，效率就会越差。

那么如果需要返回的字段很多怎么办呢？可以调整下面这个参数
```sql
show variables like 'max_length_for_sort_data';
```
这个参数限制了用于排序的行数据的长度，如果要排序的行数据的长度大于这个参数的设定值，那么排序时就不会往sort_buffer里放所有字段了，而只放要排序的列和主键。这种方式需要在排序后再通过主键去获取要返回的字段。

注意，并不是所有的ORDER BY都需要排序，使用复合索引就可以避免排序操作。

## 慢查询
```sql
show processlist;

select * from sys.schema_table_lock_waits;

select * from sys.innodb_lock_waits;

show variables like 'long_query_time';
```
查看当前语句处于什么状态，注意"waiting for table metadata lock"或"waiting for table flush"等关键字。

# 主备同步
![主备同步示意图](https://static001.geekbang.org/resource/image/a6/a3/a66c154c1bc51e071dd2cc8c1d6ca6a3.png)

1. 在备库B上通过change master命令设置主库的IP、端口、用户名、密码，以及从哪个位置开始请求binlog；
2. 在备库B上执行start slave命令，备库会启动两个线程，即io_thread和sql_thread，io_thread负责与主库建立连接；
3. 主库A校验用户名和密码后，开始按照备库B请求的位置，从本地读取binlog，通过dump_thread发送给B；
4. 备库B获得binlog后，写到本地文件relay log；
5. sql_thread读取relay log，解析日志并执行。

# 主备延迟
```sql
show slave status;
```
返回结果中*seconds_behind_master*表示备库延迟了多少秒。

主备延迟最直接的表现就是备库消费relay log的速度跟不上主库生产binlog的速度。

主备延迟可能的原因有哪些？
- 备库机器性能差
- 备库承接了大量读请求
- 大事务，比如一次删除太多数据
- 单线程复制，MySQL 5.6版本前，只支持单线程处理relay log，当主库并发高时可能出现严重的主备延迟

## 并行复制
![并行复制示意图](https://static001.geekbang.org/resource/image/bc/45/bcf75aa3b0f496699fd7885426bc6245.png)

从MySQL 5.6开始，MySQL开始支持并行复制，已经迭代了多种并行复制策略，具体暂略。

# 主备切换
## 可靠性优先策略
切换流程如下：
1. 检查备库B的延迟时间，直到小于某个值再继续下一步；
2. 把主库A修改为只读状态；
3. 检查备库B的延迟时间，直到变为0；
4. 把备库B修改为读写状态；
5. 把业务请求切到备库B。

注意，可靠性优先策略切换是有系统不可用时间的。在满足数据可靠性的前提下，MySQL高可用系统的可用性，是依赖于主备延迟的。延迟的时间越小，在主库故障的时候，服务恢复需要的时间就越短，可用性就越高。

## 可用性优先策略
1. 把备库B修改为读写状态；
2. 把业务请求切到备库B；
3. 把主库A修改为只读状态。

注意，可用性优先策略可能导致数据不一致。

## 可靠性 vs 可用性
一般的，数据的可靠性是优先于可用性的，应当在可靠性的基础上，通过降低主备延迟来提高可用性。

## 如何判断主库是不是出问题了？
select 1，不准确，select 1成功返回只能说明这个库的进程还在，并不能说明库没有问题。比如，并发查询数超出了*innodb_thread_concurrency*的限制，系统已经不可用，但select 1成功返回。

查询判断，可以检测出由于并发查询过多导致的数据库不可用，但依然不准确，比如，当binlog所在磁盘的空间占用率达到100%，那么所有的更新语句和事务提交都会被堵住，但查询数据没有问题。

更新判断，相对比较常用的方案，缺点是可能不能及时发现系统不可用。

MySQL 5.6开始提供的performance_schema库，相关统计信息也可以用于辅助判断数据库的健康状态。

# 读写分离
## 直连
![读写分离示意图](https://static001.geekbang.org/resource/image/aa/79/aadb3b956d1ffc13ac46515a7d619e79.png)

## proxy
![读写分离示意图](https://static001.geekbang.org/resource/image/1b/45/1b1ea74a48e1a16409e9b4d02172b945.jpg)