# Redis
```sh
redis-server
redis-server /xxx/redis.conf
```

## maxmemory
```
maxmemory 500000000
maxmemory-policy allkeys-lru
```
Configuring the **maxmemory** setting in the Redis configuration is crucial: if there is no cap to the maximum memory usage, the hit will eventually be 100% since all the keys can be stored in memory. If too many keys are specified with maximum memory, eventually all of the computer RAM will be used. It is also needed to configure an appropriate *maxmemory policy*, most of the time **allkeys-lru** is selected.

## save
```
save 300 100
dbfilename dump.rdb
dir ./
```

## appendonly
```
appendonly yes
appendfilename "appendonly.aof"
appendsync always/everysec/no
```

## replicaof
```
replicaof 127.0.0.1 6379
masterauth xxx
repl-backlog-size 1mb
repl-backlog-ttl 3600
min-replicas-to-write 3
min-replicas-max-lag 10
replica-read-only yes
```

# Redis-cli
External programs talk to Redis using a TCP socket and a Redis specific protocol. This protocol is implemented in the Redis client libraries for the different programming languages. 

**redis-cli** is a command line utility that can be used to send commands to Redis.
```sh
redis-cli -h 127.0.0.1 -p 6379
```

Continuous stats mode is probably one of the lesser known yet very useful features of **redis-cli** to monitor Redis instances in real time.
```sh
redis-cli --stat
```

**redis-cli** can work as a key space analyzer. It scans the dataset for big keys and provides information about the data types that the data set consists of.
```sh
redis-cli --bigkeys
```

Similar to the **bigkeys** option, **--memkeys** allows you to scan the entire keyspace to find biggest keys as well as the average sizes per key type.
```sh
redis-cli --memkeys
```

It is also possible to scan the key space in a way that does not block the Redis server (which does happen when you use command like **KEYS**), and print all the key names, or filter them for specific patterns.
```sh
redis-cli --scan
redis-cli --scan --pattern '*test*'
```

The monitoring mode is entered once you use the **MONITOR** command. All commands received by the active Redis instance will be printed to the standard output.
```sh
redis-cli MONITOR
```

Redis is often used in contexts where latency is very critical. Latency involves multiple moving parts within the application, from the client library to the network stack, to the Redis instance itself. The recis-cli has multiple facilities for studying the latency of a Redis instance and understanding the latency's maximum, average, and distribution.
```sh
redis-cli --latency
redis-cli --latency --latency-history
redis-cli --latency-dist
redis-cli --intrinsic-latency
```

During a Redis replication's first synchronization, the primary and the replica exchange the whole data set in the form of an RDB file. This feature is exploited by redis-cli in order to provide a remote backup facility that allows a transfer of an RDB file from any Redis instance to the local computer running redis-cli.
```sh
redis-cli --rdb /tmp/dump.rdb
```

The replica mode of the CLI is an advanced feature useful for Redis developers and for debugging operations. It allows for the inspection of the content a primary sends to its replicas in the replication stream in order to propagate the writes to its replicas.
```sh
redis-cli --replica
```

Redis is often used as a cache with LRU eviction. Depending on the number of keys and the amount of memory allocated for the cache (specified via the **maxmemory** directive), the amount of cache hits and misses will change. Sometimes, simulating the rate of hits is very useful to correctly provision your cache. Theoretically, given the distribution of the requests and the Redis meomory overhead, it should be possible to compute the hit rate analytically with a mathematical formula. However, Redis can be configured with different LRU settings (number of samples) and LRU's implementation, which is approximated in Redis, changes a lot between different versions. Similarly the amount of memory per key may change between versions. That is why this tool was built: it's main motivation was for testing the quality of Redis's LRU implementation, but now is also useful for testing how a given version behaves with the settings originally intended for deployment. To use this mode, specify the amount of keys in the test and configure a sensible **maxmemory** setting as a first attempt.
```sh
redis-cli --lru-test 10000000
```

# Redis Sentinel
```sh
redis-sentinel sentinel.conf
```

```
sentinel monitor mymaster 127.0.0.1 6379 2
```
*mymaster* is the name of the master set. It identifies the master and its replicas. Since each master set has a different name, Sentinel can monitor different sets of masters and replicas at the same time.

# Redis Cluster
```sh
redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1

redis-cli --cluster check 127.0.0.1:7000
```

# Redis Admin Advices
## Linux
Deploy Redis using the Linux operating system, where most of the stress testing is performed, and where most production deployments are run.

Set the Linux kernel **overcommit_memory** setting to 1.

## memory
Ensure that swap is enabled and that your swap file size is equal to amount of memory on your system. If Linux does not have swap set up, and your Redis instance accidently consumes too much memory, Redis can crash when it is out of memory, or the Linux kernel OOM killer can kill the Redis process. When swapping is enabled, you can detect latency spikes and act on them.

Set an explicit **maxmemory** option limit in your instance to make sure that it will report errors instead of failing when the system memory limit is near to be reached. Note that **maxmemory** should be set by calculating the overhead for Redis, other than data, and the fragmentation overhead. So if you think you have 10 GB of free memory, set it to 8 or 9.

If you are using Redis in a write-heavy application, while saving an RDB file on disk or rewriting the AOF log, Redis can use up to 2 times the memory normally used. The additional memory used is proportional to the number of memory pages modified by writes during the saving process, so it is often proportional to the number of keys touched during this time. Make sure to size your memory accordingly.

## replication
Set up a non-trivial replication backlog in proportion to the amount of memory Redis is using. The backlog allows replicas to sync with the master instance much more easily.

If you use replication, Redis performs RDB saves even if persistence is diabled. (This does not apply to diskless replication.) If you don't have disk usage on the master, enable diskless replication.

If you are using replication, ensure that either your master has persistency enabled, or that it does not automatically restart on crashes. Replicas will try to maintain an exact copy of the master, so if a master restarts with an empty data set, replicas will be wiped as well.

## security
By default, Redis does not require any authentication and listens to all the network interfaces. This is a big security issue if you leave Redis exposed on the internet or other places where attackers can reach it.