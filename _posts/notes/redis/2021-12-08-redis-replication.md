At the base of Redis replication there is a *leader follower* (master-replica) replication that is simple to use and configure. It allows replica Redis instances to be exact copies of master instances. The replica will automatically reconnect to the master every time the link breaks, and will attempt to be an exact copy of it regradless of what happens to the master.

This system works using three main mechanisms:
1. When a master and a replica instance are well-connected, the master keeps the replica updated be sending a stream of commands to the replica to replicate the effects on the dataset happening on the master side due to: client writes, keys expired or evicted, any other action changing the master dataset.
2. When the link between the master and the replica breaks, for network issues or because a timeout is sensed in the master or the replica, the replica reconnects and attempts to proceed with a partial resynchronization: it means that it will try to just obtain the part of the stream of commands it missed during the disconnection.
3. When a partial resynchronization is not possible, the replica will ask for a full resynchronization. This will involve a more complex process in which the master needs to create a snapshot of all its data, send it to the replica, and then continue sending the stream of commands as the dataset changes.

Redis uses by default asynchronous replication, which being low latency and high performance, is the natural replication mode for the vast majority of Redis use cases. Redis replicas asynchronously acknowledge the amount of data they received periodically with the master. So the master does not wait every time for a command to be processed by the replicas, however it knows, if needed, what replica already processed what command. This allows having optional synchronous replication. Synchronous replication of certain data can be requested by the client using the **WAIT** command.

## important facts about Redis replication:
- Redis uses asynchronous replication, with asynchronous replica-to-master acknowledges of the amount of data processed
- a master can have multiple replicas
- replicas are able to accept connections from other replicas. Aside from connecting a number of replicas to the same master, replicas can also be connected to other replicas in a cascading-like structure
- replication is non-blocking on the master side. This means that the master will continue to handle queries when one or more replicas perform the initial synchronization or a partial resynchronization
- replication is also largely non-blocking on the replica side
- replication can be used both for scalability, to have multiple replicas for read-only queries (for example, slow O(N) operations can be offloaded to replicas), or simply for improving data safety and high availability
- you can use replication to avoid the cost of having the master writing the full dataset to disk: a typical technique involves configuring your master *redis.conf* to avoid persisting to disk at all, then connect a replica configured to save from time to time, or with AOF enabled. However this setup must be handled with care, since a restarting master will start with an empty dataset: if the replica tries to sync with it, the replica will be emptied as well

## safety of replication when master has persistence turned off
In setups where Redis replication is used, it is strongly advised to have persistence turned on in the master and in the replicas. When this is not possible, for example because of latency concerns due to very slow disks, instances should be configured to avoid restarting automatically after a reboot.

Consider this scenario:
1. we have a setup with node A acting as master, with persistence turned down, and nodes B and C replicating from node A
2. node A crashes, however it has some auto-restart system, that restarts the process. However since persistence is turned off, the node restarts with an empty data set
3. nodes B and C will replicate from node A, which is empty, so they'll effectively destroy their copy of the data.

## how Redis replication works
Every Redis master has a replication ID: it is a large pseudo random string that marks a given history of the dataset. Each master also takes an offset that increments for every byte of replication stream that it is produced to be sent to replicas, to update the state of the replicas with the new changes modifying the dataset. So basically every given pair of replication ID and offset identifies an exact version of the dataset of a master.

When replicas connect to masters, they use the **PSYNC** command in order to send their old master replication ID and the offsets they processed so far. This way the master can send just the incremental part needed. However if there is not enough *backlog* in the master buffers, or if the replica is referring to an history (replication ID) which is no longer known, then a full resynchronization happens: in this case the replica will get a full copy of the dataset, from scratch.

A replication ID basically marks a given history of the data set. Every time an instance restarts from scratch as a master, or a replica is promoted to master, a new replication ID is generated for this instance. The replica connected to a master will inherit its replication ID after the handshake. So two instances with the same ID are related by the fact that they hold the same data, but potentially at a different time. It's the offset that works as a logical time to understand, for a given history (replication ID), who holds the most updated data set.

For instance, if two instances A and B have the same replication ID, but one with offset 1000 and one with offset 1023, it means that the first lacks certain commands applied to the data set. It also means that A, by applying just a few commands, may reach exactly the same state of B.

Why a replica promoted to master needs to change its replication ID after a failover? Because it is possible that the old master is still working as a master because of some network partition, retaining the same replication ID would violate the fact that the same ID and same offset of any two random instances mean they have the same data set.

Why Redis instances have two replication IDs? Because of replicas that are promoted to masters. After a failover, the promoted replica requires to still remember what was its past replication ID, because such replication ID was the one of the former master. In this way, when other replicas will synchronize with the new master, they will try to perform a partial resynchronization using the old master replication ID. This will work as expected, because when the replica is promoted to master it sets its secondary ID to its main ID,remembering what was the offset when this ID switch happened. Later it will select a new random replication ID, because a new history begins. When handling the new replicas connecting, the master will match their IDs and offsets both with the current ID and the secondary ID. In short this means that after a failover, replicas connecting to the newly promoted master don't have to perform a full sync.

Normally a full synchronization requires creating an RDB file on disk, then reloading the same RDB from disk to feed the replicas with the data. With slow disks this can be a very stressing operation for the master. Redis version 2.8.18 is the first version to have support for diskless replication. In this setup the child process directly sends the RDB over the wire to replicas, without using the disk as intermediate storage.

## read-only replica
Since Redis 2.6, replicas support a read-only mode that is enabled by default. 

Read-only replicas will reject all write commands, so that it is not possible to write to a replica because of a mistake. This does not mean that the feature is intended to expose a replica instance to the internet or more generally to a network where untrusted clients exist, because administrative commands like **DEBUG** or **CONFIG** are still enabled.

Using writable replicas can result in inconsistency between the master and the replica, so it is not recommended to use writable replicas. If you insist on using writable replicas, we suggest you follow these recommendations:
- don't write to keys in a writable replica that are also used on the master
- don't configure an instance as a writable replica as an intermediary step when upgrading a set of instances in a running system. In general, don't configure an instance as a writable replica if it can ever be promoted to a master if you want to guarantee data consistency

## allow writes only with N attached replicas
Starting with Redis 2.8, you can configure a Redis master to accept write queries only if at least N replicas are currently connected to the master. However, because Redis uses asynchronous replication it is not possible to ensure the replica actually received a given write, so there is always a window for data loss.

This is how the feature works:
- Redis replicas ping the master every second, acknowledging the amount of replication stream processed
- Redis masters will remember the last time it received a ping from every replica
- the user can configure a minimum number of replicas that have a lag not greater than a maximum number of seconds

If there are at least N replicas, with a lag less than M seconds, then the write will be accepted. If the conditions are not met, the master will instead reply with an error and the write will not be accepted.

You may think of it as a best effort data safety mechanism, where consistency is not ensured for a given write, but at least the time window for data loss is restricted to a given number of seconds. In general bound data loss is better than unbound one.

## maxmemory on replicas
By default, a replica will ignore **maxmemory** (unless it is promoted to master after a failover or manually). It means that the eviction of keys will be handled by the master, sending the **DEL** commands to the replica as keys evict in the master side.

Note that since the replica by default does not evict, it may end up using more memory than what is set via **maxmemory**. Make sure you monitor your replicas, and make sure they have enough memory to never hit a real out-of-memory condition before the master hits the configured **maxmemory** setting.

## replicas priority
Redis instances have a configuration parameter called **replica-priority**. This information is exposed by Redis replica instances in their **INFO** output, and Sentinel uses it in order to pick a replica among the ones that can be used in order to failover a master. 

If the replica priority is set to 0, the replica is never promoted to master. Replicas with a lower priority number are preferred by Sentinel.