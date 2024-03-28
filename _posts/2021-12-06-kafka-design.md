# Persistence
Kafka relies heavily on the filesystem for storing and caching messages. There is a general perception that "disks are slow" which make people skeptical that a persistent structure can offer competitive performance. In fact disks are both much slower and much faster than people expect depending on how they are used; and a properly designed disk structure can often be as fast as the network.

Having access to virtually unlimited disk space without any performance penalty means that we can provide some features not usually found in a messaging system. For example, in Kafka, instead of attempting to delete messages as soon as they are consumed, we can retain messages for a relatively long period. This leads to a great deal of flexibility for consumers.

# Efficiency
Once poor disk access patterns have been eliminated, there are two common causes of inefficiency in this type of system: too many small I/O operations, and excessive byte copying.

The small I/O problem happens both between the client and the server and in the server's own persistent operations. To avoid this, our protocol is built around a "message set" abstraction that naturally groups messages together. This allows network requests to group messages together and amortize the overhead of the network roundtrip rather than sending a single message at a time. The server in turn appends chunks of messages to its log in one go, and the consumer fetches large linear chunks at a time. This simple optimization produces orders of magnitude speed up. Batching leads to larger network packets, larger sequential disk operations, contiguous memory blocks, and so on, all of which allows Kafka to turn a bursty stream of random messages writes into linear writes that flow to the consumers.

The other inefficiency is in byte copying. At low message rates this is not an issue, but under load the impact is significant. To avoid this we employ a standardized binary message format that is shared by the producer, the broker, and the consumer (so data chunks can be transferred without modification between them). Maintaining this common format allows optimization of the most important operation: network transfer of persistent log chunks. Modern unix operating systems offer a highly optimized code path for transferring data out of pagecache to a socket; in Linux this is done with the sendfile system call.

To understand the impact of sendfile, it is important to understand the common data path for transfer of data from file to socket:
1. the operating system reads data from the disk into pagecache in kernel space
2. the application reads the data from kernel space into a user-space buffer
3. the application writes the data back into kernel space into a socket buffer
4. the operating system copies the data from the socket buffer to the NIC buffer where it is sent over the network

Using sendfile, this re-copying is avoided by allowing the OS to send the data from pagecache to the network directly.

Using the zero-copy optimization, data is copied into pagecache exactly once and reused on each consumption instead of being stored in memory and copied out to user-space every time it is read. This allows messages to be consumed at a rate that approaches the limit of the network connection. 

This combination of pagecache and sendfile means that on a Kafka cluster where the consumers are mostly caught up you will see no read activity on the disks whatsoever as they will be serving data entirely from cache.

In some cases the bottleneck is actually not CPU or disk but network bandwidth. Of course, the user can always compress its messages one at a time without any support needed from Kafka, but this can lead to poor compression ratios as much of the redundancy is due to repetition between messages of the same type. Efficient compression requires compressing multiple messages together rather than compressing each message individually. Kafka supports this with an efficient batching format. A batch of messages can be grouped together, compressed, and sent to the server in this form. The broker decompresses the batch in order to validate it. This batch of messages is then written to disk in compressed form. The batch will remain compressed in the log and it will also be transmitted to the consumer in compressed form. The comsumer decompresses any compressed data that it receives.

# The Producer
The producer sends data directly to the broker that is the leader for the partition without any intervening routing tier. To help the producer do this all Kafka nodes can answer a request for metadata about which servers are alive and where the leaders for the partitions of a topic are at any given time to allow the producer to appropriately direct its requests.

The client controls which partition it publishes messages to. This can be done at random, implementing a kind of random load balancing, or it can be done by some semantic partitioning function.

Batching is one of the big drivers of efficiency, and to enable batching the Kafka producer will attempt to accumulate data in memory and to send out larger batches in a single request. The batching can be configured to accumulate no more than a fixed number of messages and to wait no longer than some fixed latency bound (say 64k or 10ms). This allows the accumulation of more bytes to send, and fewer larger I/O operations on the servers.

# Consumer
The Kafka consumer works by issuing "fetch" requests to the brokers leading the partitions it wants to consume. The consumer specifies its offset in the log with each request and receives back a chunk of log beginning from that position. The consumer thus has significant control over this position and can rewind it to re-consume data if need be.

## push vs. pull
Kafka follows a traditional design, shared by most messaging systems, where data is pushed to brokers from the producer and pulled from the broker by the consumer. A push-based system has difficulty dealing with diverse consumers as the broker controls the rate at which data is transferred. Another advantage of a pull-based system is that it lends itself to aggressive batching of data sent to the consumer.

## consumer position
Keeping track of what has been consumed is, surprisingly, one of the key performance points of a messaging system. 

Most messaging systems keep metadata about what messages have been consumed on the broker. That is, as a message is handed out to a consumer, the broker either records the fact locally immediately or it may wait for acknowledgement from the consumer.

What is perhaps not obvious is that getting the broker and consumer to come into agreement about what has been consumed is not a trivial problem. Tricky problems must be dealt with, like what to do with messages that are sent but never acknowledged.

Kafka handles this differently. Our topic is divided into a set of totally ordered partitions, each of which is consumed by exactly one consumer within each subscribing consumer group at any given time. This means that the position of a consumer in each partition is just a single integer, the offset of the next message to consume. This makes the state about what has been consumed very small, just one number for each partition. This state can be periodically checkpointed. This makes the equivalent of message acknowledgements very cheap.

There is a side benefit of this decision. A consumer can deliberately rewind back to an old offset and re-consume data.

# Message Delivery Semantics
There are multiple possible message delivery guarantees that could be provided:
- at most once, messages may be lost but are never redelivered
- at least once, messages are never lost but may be redelivered
- exactly once, this is what people actually want, each message is delivered once and only once

It breaks down into two problems: the durability guarantees for publishing a message and the guarantees when consuming a message.

When publishing a message Kafka have a notion of the message being "committed" to the log. Once a published message is committed it will not be lost as long as one broker that replicates the partition to which this message was written remains "alive". If a producer attempts to publish a message and experiences a network error it cannot be sure if this error happened before or after the message was committed. This is similar to the semantics of inserting into a database table with an autogenerated key. Prior to 0.11.0.0, if a producer failed to receive a response indicating that a message was committed, it had little choice but to resend the message. Since 0.11.0.0, the Kafka producer also supports an idempotent delivery option which guarantees that resending will not result in duplicate entries in the log. For uses which are latency sensitive we allow the producer to specify the durability level it desires. If the producer specifies that it wants to wait on the message being committed this can taken on the order of 10 ms. However the producer can also specify that it wants to perform the send completely asynchronously or that it wants to wait only until the leader have the message.

# Replication
Kafka replicates the log for each topic's partitions across a configurable number of servers. This allows automatic failover to these replicas when a server in the cluster fails so messages remain available in the presence of failures.

The unit of replication is the topic partition. Under non-failure conditions, each partition in Kafka has a single leader and zero or more followers. The total number of replicas including the leader constitute the replication factor. All writes go to the leader of the partition, and reads can go to the leader or the followers of the partition. Typically, there are many more partitions than brokers and the leaders are evenly distributed among brokers. The log on the followers are identical to the leader's log, all have the same offsets and messages in the same order (though, of course, at any given time the leader may have a few as-yet unreplicated messages at the end of its log).

Followers consume messages from the leader just as a normal Kafka consumer would and apply them to their own log. Having the followers pull from the leader has the nice property of allowing the follower to naturally batch together log entries they are applying to their log.

As with most distributed systems, automatically handling failures requires a precise definition of what it means for a node to be "alive". In Kafka, a special node known as the "controller" is responsible for managing the registration of brokers in the cluster. Broker liveness has two conditions:
- brokers must maintain an active session with the controller in order to receive regular metadata updates
- brokers acting as followers must replicate the writes from the leader and not fall "too far" behind

We refer to nodes satisfying these two conditions as being "in sync" to avoid the vagueness of "alive" or "failed". The leader keeps track of the set of "in sync" replicas, which is known as the ISR. If either of these conditions fail to be satisfied, then the broker will be removed from the ISR.

We can now more precisely define that a message is considered committed when all replicas in the ISR for that partition have applied it to the their log. Only committed message are ever given out to the consumer. This means that the consumer need not worry about potentially seeing a message that could be lost if the leader fails. Producers, on the other hand, have the option of either waiting for the message to be committed or not, depending on their preference for tradeoff between latency and durability. This preference is controlled by the acks setting that the producer uses. Note that topics have a setting for the "minimum number" of in-sync replicas that is checked when the producer requests acknowledgement that a message has been written to the full set of in-sync replicas. If a less stringent acknowledgement is requested by the producer, then the message can be committed, and consumed, even if the number of in-sync replicas is lower than the minimum (e.g. it can be as low as just the leader).

A replicated log models the process of coming into consensus on the order of a series of values. There are many ways to implement this, but the simplest and fastest is with a leader who chooses the ordering of values provided to it. As long as the leader remains alive, all followers need to only copy the values and ordering the leader chooses.

When the leader does die we need to choose a new leader from among the followers. But followers may fall behind or crash so we must ensure we choose an up-to-date follower. The fundamental guarantee a log replication algorithm must provide is that if we tell the client a message is committed, and the leader fails, the new leader we elect must also have that message. This yields a tradeoff: if the leader waits for more followers to acknowledge a message before declaring it committed then there will be more potentially electable leaders.

If you choose the number of acknowledgements required and the number of logs that must be compared to elect a leader such that there is guaranteed to be an overlap, then this is called a Quorum.

A common approach to this tradeoff is to use a majority vote for both the commit decision and the leader election. Kafka takes a slightly different approach to choosing its quorum set. Instead of majority vote, Kafka dynamically maintains a set of in-sync replicas (ISR) that are caught-up to the leader. Only members of this set are eligible for election as leader. A write to a Kafka partition is not considered committed until all in-sync replicas have received the write. With this ISR model and f+1 replicas, a Kafka topic can tolerate f failures without losing committed messages.

A practical system needs to do something reasonable when all the replicas die. If you are unlucky enough to have this occur, it is important to consider what will happen. There are two behaviors that could be implemented:
- wait for a replica in the ISR to come back to life and choose this replica as the leader 
- choose the first replica that comes back to life as the leader

This is a simple tradeoff between availability and consistency. By default from version 0.11.0.0, Kafka chooses the first strategy and favor waiting for a consistent replica.

# Log Compaction
Log compaction ensures that Kafka will always retain at least the last known value for each message key within the log data for a single topic partition. It addresses use cases and scenarios such as restoring state after application crashes or system failure, or reloading caches after application restarts during operational maintenance.

Log compaction gives us a more granular retention mechanism so that we are guaranteed to retain at least the last update for each primary key. By doing this we guarantee that the log contains a full snapshot of the final value for every key not just keys that changed recently. This means downstream consumers can restore their own state off this topic without us having to retain a complete log of all changes.

Log compaction is a mechanism to give finer-grained per-record retention, rather than the coarser-grained time-based retention. The idea is to selectively remove records where we have a more recent update with the same primary key. This way the log is guaranteed to have at least the last state for each key.