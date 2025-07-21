Redis Cluster provides a way to run a Redis installation where data is automatically sharded across multiple Redis nodes. Redis Cluster also provides some degree of availability during partitions, in practical terms, the ability to continue operations when some nodes fail or are unable to communicate. However, the cluster will become unavailable in the event of larger failures (for example, when the majority of masters are unavailable).

What do you get with Redis Cluster?
- automatically split your dataset among multiple nodes
- continue operations when a subset of the nodes are experiencing failures or are unable to communicate with the rest of the cluster

## Redis Cluster TCP ports
Every Redis Cluster node requires two open TCP connections: a Redis TCP port used to serve clients and second port known as the *cluster bus port*.

Cluster bus is a node-to-node communication channel that uses a binary protocol, which is more suited to exchanging information between nodes due to little bandwidth and process time. Nodes use the cluster bus for failure detection, configuration updates, failover authorization, and so forth.

If you dont't open both TCP ports, you cluster will not work as expected. For a Redis Cluster to work properly you need, for each node:
- the client communication port (usually 6379) used to communicate with clients and be open to all the clients that need to reach the cluster, plus all the other cluster nodes that use the client port for key migrations
- the cluster bus port must be reachable from all the other cluster nodes

## Redis Cluster data sharding
Redis Cluster does not use consistent hashing, but a different form of sharding where every key is conceptually part of what we call a **hash slot**.

There are 16384 hash slots in Redis Cluster, and to compute the hash slot of a given key, we simply take the CRC16 of the key modulo 16384.

Every node in a Redis Cluster is responsible for a subset of the hash slots, so, for example, you may have a cluster with 3 nodes, where:
- node A contains hash slots from 0 to 5500
- node B contains hash slots from 5501 to 11000
- node C contains hash slots from 11001 to 16383

This makes it easy to add and remove cluster nodes. Moving hash slots from a node to another does not require stopping any operations; therefore, adding and removing nodes, or changing the percentage of hash slots hold by nodes, requires no downtime.

Redis Cluster supports multiple key operations as long as all of the keys involved in a single command execution (or whole transaction, or Lua script execution) belong to the same hash slot. The user can force multiple keys to be part of the same hash slot by using a feature called *hash tags*. Simply speaking, if there is a substring between {} brackets in a key, only what is inside the string is hashed. For example, the key user:{123}:profile and user:{123}:account are guaranteed to be in the same hash slot because they share the same hash tag.

Resharding basically means to move hash slots from a set of nodes to another set of nodes, and like cluster creation it is accomplished using the redis-cli utility.

## Redis Cluster master-replica model
To remain available when a subset of cluster nodes are failing or are not able to communicate with the majority of nodes, Redis Cluster uses a master-replica model where every hash slot has from 1 (the master itself) to N replicas (N-1 additional replica nodes).

For example, we may have a cluster composed of A, B, C that are master nodes, and A1, B1, C1 that are replica nodes. Node B1 replicates B, and when B fails, the cluster will promote node B1 as the new master and will continue to operate correctly. However, note that if nodes B and B1 fail at the same time, Redis Cluster will not be able to continue to operate.

Note that the minimal cluster that works as expected must contain at least three master nodes. For deployment, we strongly recommend a six-node cluster, with three masters and three replicas.

## Redis Cluster consistency guarantees
Redis Cluster does not guarantee **strong consistence**. In practical terms this means that under certain conditions, it is possible that Redis Cluster will lose writes that were acknowledged by the system to the client.

The first reason why Redis Cluster can lose writes is because it uses asynchronous replication. This means that during writes the following happens:
- your client writes to the master A.
- the master A replies OK to your client.
- the master A propagates the writes to its replicas A1, A2, A3.

As you can see, A does not wait for an acknowledgement from A1, A2, A3 before replying to the client, since this would be a prohibitive latency penalty for Redis, so if your client writes something, A acknowledges the write, but crashes before being able to send the write to its replicas, one of the replicas (that did not receive the write) can be promoted to master, losing the write forever.

This is very similar to what happens with most databases that are configured to flush data to disk every second. Basically, there is a trade-off to be made between performance and consistency.

Redis Cluster has support for synchronous writes when absolutely needed, implemented via the **WAIT** command. This makes losing writes a lot less likely. However, note that Redis Cluster does not implement strong consistency even when synchronous replication is used: it is always possible, under more complex failure scenarios, that a replica that was not able to receive the write will be elected as master.

There is another notable scenario where Redis Cluster will lose writes, that happens during a network partition where a client is isolated with a minority of instances including at least a master.

## node ID
Node ID is used forever by the specific instance in order for the instance to have a unique name in the context of the cluster. Every node remembers every other node using this IDs, and not by IP or port. IP addresses and ports may change, but the unique node identifier will never change for all the life of the node.

## redis-cli cluster support
```sh
redis-cli -c -p 7000
```

The redis-cli cluster support is very basic, so it always uses the fact that Redis Cluster nodes are able to redirect a client to the right node. A serious client is able to do better than that, and cache the map between hash slots and node addresses, to directly use the right connection to the right node. The map is refreshed only when something changed in the cluster configuration, for example after a failover or after the system administrator changed the cluster layout by adding or removing nodes.

## reshard the cluster
Resharding basically means to move hash slots from a set of nodes to another set of nodes. You only need to specify a single node, redis-cli will find the other nodes automatically.

```sh
redis-cli --cluster reshard 127.0.0.1:7000

redis-cli --cluster reshard 127.0.0.1:7000 --cluster-from from_node_id --cluster-to to_node_id --cluster-slots number_of_slots --cluster-yes
```

## failover
To trigger the failover, the simplest thing we can do (that is also the semantically simplest failure that can occur in a distributed system) is to crash a single process, in our case a single master.

```sh
redis-cli -p 7002 debug segfault
```

Sometimes it is useful to force a failover without actually causing any problem on a master. For example, to upgrade the Redis process of one of the master nodes it is a good idea to failover it to turn it into a replica with minimal impact on availability. Manual failures are supported by Redis Cluster using the **CLUSTER FAILOVER** command, that must be executed in one of the replicas of the master you want to failover.

Manual failovers are special and are safer compared to failovers resulting from actual master failures. They occur in a way that avoids data loss in the process, by switching clients from the original master to the new master only when the system is sure that the new master processed all the replication stream from the old one.

Basically clients connected to the master we are failing over are stopped. At the same time the master sends its replication offset to the replica, that waits to reach the offset on its side. When the replication offset is reached, the failover starts, and the old master is informed about the configuration switch. When the clients are unblocked on the old master, they are redirected to the new master.

To promote a replica to master, it must first be known as a replica by a majority of the masters in the cluster. Otherwise, it cannot win the failover election. If the replica has just been added to the cluster, you may need to wait a while before sending the **CLUSTER FAILOVER** command, to make sure the masters in cluster are aware of the new replica.

## add a new node
Adding a new node is basically the process of adding an empty node and then moving some data into it, in case it is a new master, or telling it to setup as a replica of a known node, in case it is a replica.

```sh
redis-cli --cluster add-node new_node_ip:new_node_port one_cluster_node_ip:one_cluster_node_port
```
After adding the new master, it is possible to assign hash slots to this node using the resharding feature of redis-cli.

```sh
redis-cli --cluster add-node new_node_ip:new_node_port one_cluster_node_ip:one_cluster_node_port --cluster-slave
```
A more manual way to add a replica to a specific master is to add the new node as an empty master, and then turn it into a replica using the **CLUSTER REPLICATE** command. This also works if the node was added as a replica but you want to move it as a replica of a different master.

## remove a node
```sh
redis-cli --cluster del-node one_cluster_node_ip:one_cluster_node_port to_delete_node_id
```

You can remove a master node in the same way as well, however in order to remove a master node it must be empty. If the master is not empty you need to reshard data away from it to all the other master nodes before.

There is a special scenario where you want to remove a failed node. You should not use the del-node command because it tries to connect to all nodes and you will encounter a connection refused error. You can use the call command:
```sh
redis-cli --cluster call one_cluster_node_ip:one_cluster_node_port cluster forget failed_node_id
```

## replica migration
There is a special scenario where you want replicas to move from one master to another one automatically, without the help of the system administrator. The automatic reconfiguration of replicas is called *replica migration* and is able to improve the reliability of a Redis Cluster.

The reason why you may want to let your cluster replicas to move from one master to another under certain condition, is that usually the Redis Cluster is as resistant to failures as the number of replicas attached to a given master. To imporve reliability of the system we have the option to add additional replicas to every master, but this is expensive. Replica migration allows to add more replicas to just a few masters. With replica migration what happens is that if a master is left without replicas, a replica from a master that has multiple replicas will migrate to the orphaned master.

What you should know about replicas migration in short?
- the cluster will try to migrate a replica from the master that has the greatest number of replicas in a given moment
- to benefit from replica migration you have just to add a few more replicas to a single master in your cluster, it does not matter what master
- there is a configuration parameter that controls the replica migration feature that is called **cluster-migration-barrier**

## upgrade nodes in a Redis Cluster
Upgrading replica nodes is easy since you just need to stop the node and restart it with an updated version of Redis.

Upgrading masters is a bit more complex, and the suggested procedure is:
- use **CLUSTER FAILOVER** to trigger a manual failover of the master to one of its replicas
- wait for the master to turn into a replica
- upgrade the node as you do for replicas
- if you want the master to be the node you just upgraded, trigger a new manual failover in order to turn back the upgraded node into a master

## migrate to Redis Cluster