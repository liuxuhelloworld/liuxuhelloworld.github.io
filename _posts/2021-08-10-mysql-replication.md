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

# 读写分离
## 直连
![读写分离示意图](https://static001.geekbang.org/resource/image/aa/79/aadb3b956d1ffc13ac46515a7d619e79.png)

## proxy
![读写分离示意图](https://static001.geekbang.org/resource/image/1b/45/1b1ea74a48e1a16409e9b4d02172b945.jpg)

## 主从延迟
- 强制走主库

专栏里介绍的其他几种方式，有的不可靠，有的我觉得太复杂，如果有专门的中间件团队研发维护可能还好，这里就暂略了。