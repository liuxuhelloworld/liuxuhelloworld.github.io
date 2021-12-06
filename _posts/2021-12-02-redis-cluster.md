# What is Redis Cluster?
Redis Cluster provides a way to run a Redis installation where data is automatically sharded across multiple Redis nodes.

What do you get with Redis Cluster?
- the ability to automatically split your dataset among multiple nodes.
- the ability to continue operations when a subset of the nodes are experiencing failures or are unable to communicate with the rest of the cluster.

# Redis Cluster TCP ports
Every Redis Cluster node requires two TCP connections open. The normal Redis TCP port used to serve clients and the second port named cluster bus port.

The cluster bus port is used for the Cluster bus, that is a node-to-node communication channel using a binary protocol. The Cluster bus is used by nodes for failure detection, configuration update, failover authorization, and so forth.

# Redis Cluster data sharding
Redis Cluster does not use consistent hashing, but a different form of sharding where every key is conceptually part of what we call a **hash slot**.

There are 16384 hash slots in Redis Cluster, and to compute what is the hash slot of a given key, we simply take the CRC16 of the key modulo 16384.

Every node in a Redis Cluster is responsible for a subset of the hash slots. This allows to add and remove nodes in the cluster easily. Because moving hash slots from a node to another does not require to stop operations, adding and removing nodes, or changing the percentage of hash slots hold by nodes, does not require any downtime.

## resharding
Resharding basically means to move hash slots from a set of nodes to another set of nodes, and like cluster creation it is accomplished using the redis-cli utility.

# Redis Cluster master-replica model
In order to remain available when a subset of cluster nodes are failing or are not able to communicate with the majority of nodes, Redis Cluster uses a master-replica model.

# Redis Cluster consistency guarantees
Redis Cluster is not able to guarantee **strong consistence**. In practical terms this means that under certain conditions, it is possible that Redis Cluster will lose writes that were acknowledged by the system to the client.

The first reason why Redis Cluster can lose writes is because it uses asynchronous replication. This means that during writes the following happens:
- Your client writes to the master A.
- The master A replies OK to your client.
- The master A propagates the writes to its replicas A1, A2, A3.

As you can see, A does not wait for an acknowledgement from A1, A2, A3 before replying to the client, so if your client writes something, A acknowledges the write, but crashes before being able to send the write to its replicas, one of the replicas (that did not receive the write) can be promoted to master, losing the write forever.

This is very similar to what happens with most databases that are configured to flush data to disk every second. Basically, there is a trade-off to be made between performance and consistency.

However, even when synchronous replication is used, Redis Cluster still does not guarantee strong consistency. It is always possible, under more complex failure scenarios, that a replica that was not able to receive the write will be elected as master. For example, a network partition where a client is isolated with a minority of instances including at least a master.

# Redis Cluster admin
Note that the minimal cluster requires to contain as least three master nodes.

```bash
redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1

redis-cli -c -p 7000

# nodes info
redis-cli -p 7004 cluster nodes

# reshard
redis-cli --cluster reshard 127.0.0.1:7000

# check
redis-cli --cluster check 127.0.0.1:7000
```