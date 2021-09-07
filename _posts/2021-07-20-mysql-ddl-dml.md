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

按效率粗略排序的话，
$$
count(字段) < count(主键) < count(1) \approx count(*)
$$

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