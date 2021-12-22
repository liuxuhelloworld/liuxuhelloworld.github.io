1. When a master and a replica instances are well-connected, the master keeps the replica updated be sending a stream of commands to the replica, in order to replicate the effects on the dataset happening in the master side due to: client writes, keys expired or evicted, any other action changing the master dataset.
2. When the link between the master and the replica breaks, for network issues or because a timeout is sensed in the master or the replica, the replica reconnects and attempts to proceed with a partial resynchronization: it means that it will try to just obtain the part of the stream of commands it missed during the disconnection.
3. When a partial resynchronization is not possible, the replica will ask for a full resynchronization. This will involve a more complex process in which the master needs to create a snapshot of all its data, send it to the replica, and then continue sending the stream of commands as the dataset changes.

Redis uses by default asynchronous replication, which being low latency and high performance, is the natural replication mode for the vast majority of Redis use cases. Redis replicas asynchronously acknowledge the amount of data they received periodically with the master. So the master does not wait every time for a command to be processed by the replicas, however it knows, if needed, what replica already processed what command. This allows having optional synchronous replication.

- a master can have multiple replicas
- replicas can be connected to other replicas in a cascading-like structure
- replication is non-blocking on the master side. This means that the master will continue to handle queries when one or more replicas perform the initial synchronization or a partial resynchronization
- replication is also largely non-blocking on the replica side
- it is possible to use replication to avoid the cost of having the master writing the full dataset to disk: a typical technique involves configuring your master redis.conf to avoid persisting to disk at all, then connect a replica configured to save from time to time, or with AOF enabled. However this setup must be handled with care, since a restarting master will start with an empty dataset: if the replica tries to synchronize with it, the replica will be emptied as well

In setups where Redis replication is used, it is strongly advised to have persistence turned on in the master and in the replicas. When this is not possible, for example because of latency concerns due to very slow disks, instances should be configured to avoid restarting automatically after a reboot.

Every Redis master has a replication ID: it is a large pseudo random string that marks a given history of the dataset. Each master also takes an offset that increments for every byte of replication stream that it is produced to be sent to replicas, in order to update the state of the replicas with the new changes modifying the dataset. So basically every given pair of replication ID and offset identifies an exact version of the dataset of a master.

When replicas connect to masters, they use the **PSYNC** command in order to send their old master replication ID and the offsets they processed so far. This way the master can send just the incremental part needed. However if there is not enough *backlog* in the master buffers, or if the replica is referring to an history (replication ID) which is no longer known, then a full resynchronization happens: in this case the replica will get a full copy of the dataset, from scratch.

A replication ID basically marks a given history of the data set. Every time an instance restarts from scratch as a master, or a replica is promoted to master, a new replication ID is generated for this instance. The replica connected to a master will inherit its replication ID after the handshake. So two instances with the same ID are related by the fact that they hold the same data, but potentially at a different time. 

The reason why Redis instances have two replication IDs is because of replicas that are promoted to masters. After a failover, the promoted replica requires to still remember what was its past replication ID, because such replication ID was the one of the former master. In this way, when other replicas will synchronize with the new master, they will try to perform a partial resynchronization using the old master replication ID.

Why a replica promoted to master needs to change its replication ID after a failover? It is possible that the old master is still working as a master because of some network partition: retaining the same replication ID would violate the fact that the same ID and same offset of any two random instances mean they have the same data set.

```
replicaof 127.0.0.1 6379

masterauth xxx

min-replicas-to-write <number of replicas>
min-replicas-max-lag <number of seconds>
```

Since Redis 2.6, replicas support a read-only mode that is enabled by default. Read-only replicas will reject all write commands, so that it is not possible to write to a replica because of a mistake.