# Strings
Redis strings store sequences of bytes, including text, serialized objects, and binary arrays.

By default, a single Redis string can be a maximum of 512 MB.

Most string operations are O(1), which means they're highly efficient. However, be careful with **SUBSTR**, **GETRANGE**, and **SETRANGE** commands, which can be O(n). These random-access string commands may cause performance issues when dealing with large strings.

# Lists
Redis lists are linked lists of string values. Redis lists are implemented with linked Lists because for a database system it is crucial to be able to add elements to a very long list in a very fast way. When fast access to the middle of a large collection of elements is important, you should use sorted sets.

Lists have a special feature that make them suitable to implement queues, and in general as a building block for inter process communication systems: blocking operations.

Lists are useful for a number of tasks, two very representative use cases are the following:
- remember the latest updates posted by users into a social network
- communication between processes, using a producer-consumer pattern where producer pushes items into a list, and a consumer comsumes those items and executed actions

List operations that access its head or tail are O(1), which means they're highly efficient. However, commands that manipulate elements within a list are usually O(n).

# Sets
A Redis set is an unordered collection of unique strings. You can use Redis sets to efficiently:
- track unique items
- represent relations
- perform common set operations such as intersection, unions, and differences

Most set operations, including adding, removing, and checking whether an item is a set member, are O(1). This means that they're highly efficient. However, for large sets with hundreds of thousands of members or more, you should exercise caution when running the **SMEMBERS** command. This command is O(N) and returns the entire set in a single response. As an alternative, consider the **SSCAN**, which lets you retrieve all members of a set iteratively.

# Sorted Sets
A Redis sorted set is a collection of unique strings ordered by an associated score. When more than one string has the same score, the strings are ordered lexicographically.

Sorted sets are implemented via a dual-ported data structure containing both a skip list and a hash table. Most sorted set operations are O(log(n)), where *n* is the number of members. Exercise some caution when running the **ZRANGE** command with large return values. This command's time complexity is O(log(n)+m), where *m* is the number of results returned.

Some use cases for sorted sets include:
- leaderborads, for example, you can use sorted sets to easily maintain ordered lists of the highest scores in a massive online game
- rate limiters, for example, you can use a sorted set to build a sliding-window rate limiter to prevent excessive API requests

# Hashes
Redis hashes are record types structured as collections of field-value pairs.

It's worth noting that small hashes (i.e., a few elements with small values) are encoded in special way in memory that make them very memory efficient.

Most Redis hash commands are O(1). A few commands, such as **HKEYS**, **HVALS**, **HGETALL**, and most of the expiration-related commands, are O(n), where n is the number of field-value pairs.

# Streams
A Redis stream is a data structure that acts like an append-only log but also implements several operations to overcome some of the limits of a typical append-only log. These include random access in O(1) time and complex consumption strategies, such as consumer groups.

Redis generates a unique ID for each stream entry. You can use these IDs to retrieve their associated entries later or to read and process all subsequent entries in the stream. The fact that each stream entry has an ID is another similarity with log files, where line numbers, or the byte offset inside the file, can be used in order to identify a given entry.

Adding an entry to a stream is O(1). Accessing any single entry is O(n), where *n* is the length of the ID. Since stream IDs are typically short and of a fixed length, this effectively reduces to a constant time lookup. For details on why, note that streams are implemented as radix trees.

Examples of Redis stream use cases include:
- event sourcing, e.g., tracking user actions
- sensor monitoring, e.g., reading from the devices in the field
- notifications

## query by range
We could see a stream not only as a messaging system, but also a time series store. In this case, one natural query mode is to get messages by ranges of time, or alternatively to iterate the messages using a cursor to incrementally check all the history.

To query the stream by range we are only required to specify two IDs or two timestamps.

```bash
XRANGE race:france - +
XRANGE race:france 1692632086369 1692632086371
XRANGE race:france (1692632094485-0 + COUNT 2
XREVRANGE race:france + - COUNT 1
```

## subscribe to new items
Usually what we want is to subscribe to new items arriving to the stream. Using the traditional terminology we want the streams to be able to *fan out* messages to multiple clients. 

This concept may appear related to Redis Pub/Sub, where you subcribe to a channel, or to Redis blocking lists, where you wait for a key to get new elements to fetch, but there are fundamental differences in the way you consume a stream:
- a stream can have multiple consumers waiting for data. Every new item, by default, will be delivered to every consumer that is waiting for data in a given stream. This behavior is different than blocking lists, where each consumer will get a different element. However, the ability to fan out to multiple consumers is similar to Pub/Sub
- while in Pub/Sub messages are fired and forget and are never stored anyway, and while when using blocking lists, when a message is received by the client it is popped from the list, streams work in a fundamentally different way. All the messages are appended in the stream indefinately (unless the user explicitly asks to delete entries)
- stream consumer groups provide a level of control that Pub/Sub or blocking lists cannot achieve

```bash
XREAD COUNT 2 STREAMS race:france 0
XREAD BLOCK 0 STREAMS race:france $
```

The blocking form of **XREAD** is able to listen to multiple streams, just by specifying multiple key names. If the request can be serverd synchronously because there is at least one stream with elements greater than the corresponding ID we specified, it returns with the results. Otherwise, the command will block and will return the items of the first stream which gets new data. 

Similarly to blocking list operations, blocking streams reads are *fair* from the point of view of clients waiting for data, since the semantics is FIFO style. The first client that blocked for a given stream will be the first to be unblocked when new items are available.

## consumer groups
In certain problems what we want to do is not to provide the same stream of messages to many clients, but to provide a different subset of messages from the same stream to many clients. An obvious case when this is useful is that of messages which are slow to process: the ability to have N different workers that will receive different parts of the stream allows us to scale message processing, by routing different messages to different workers that are ready to do more work.

In order to achieve this, Redis uses a concept called *consumer groups*. A consumer group is like a *pseudo consumer* that gets data from a stream, and actually serve multiple consumers, providing certain guarantees:
- each message is served to a different consumer so that it is not possible that the same message will be delivered to multiple consumers
- consumers are identified, within a consumer group, by a name, which is case-sensitive string that the clients implementing consumers must choose
- each consumer group has the concept of the *first ID never consumed* so that, when a consumer asks for new messages, it can provide just messages that were not previously delivered
- consuming a message requires an explicit acknowledgement using a specific command. Redis interprets the acknowledgement as: this message was correctly processed so it can be evicted from the consumer group
- a consumer group tracks all the messages that are currently pending, that is, messages that were delivered to some consumer of the consumer group, but are yet to be acknowledged as processed

```bash
XGROUP CREATE race:france france_riders $
XGROUP CREATE race:france france_riders 0
XREADGROUP GROUP france_riders Alice COUNT 1 STREAMS race:france >
XREADGROUP GROUP france_riders Alice COUNT 1 STREAMS race:france 0
```

In the real world consumers may permanently fail and never recover. What happens to the pending messages of the consumer that never recovers after stopping for any reason? Redis consumer groups offer a feature that is used in these situations in order to *claim* the pending messages of a given consumer so that such messages will change ownership and will be re-assigned to a different consumer.

```bash
XPENDING race:france france_riders
XPENDING race:france france_riders - + 1
XCLAIM race:france france_riders Bob 60000 1721014050450-0
XAUTOCLAIM race:france france_riders Alice 60000 0-0
```

The **XINFO** command is an observability interface that can be used with sub-commands in order to get information about streams or consumer groups.

```bash
XINFO STREAM race:france
XINFO GROUPS race:france
XINFO CONSUMERS race:france france_riders
```

Consumer groups in Redis streams may resemble in some way Kafka partitioning-based consumer groups, however note that Redis streams are, in practical terms, very different. Basically Kafka partitions are more similar to using N different Redis keys, while Redis consumer groups are a server-side load balancing system of messages from a given stream to N different consumers.

## capped streams
```bash
XADD race:italy MAXLEN 2 * rider Jones
XADD race:italy MAXLEN ~ 1000 * rider Wood

XTRIM race:italy MAXLEN 10
XTRIM mystream MAXLEN ~ 10
```

# Bitmaps
Bitmaps are not an actual data type, but a set of bit-oriented operations defined on the String type. Since strings are binary safe blobs and their maximum length is 512 MB, they are suitable to set up to 2^32 different bits.

One of the biggest advantages of bitmaps is that they often provide extreme space savings when storing information. For example in a system where different users are represented by incremental user IDs, it is possible to remember a single bit information of 4 billion of users using just 512 MB of memory.

Bitmaps are trivial to split into multiple keys, for example for the sake of sharding the data set abd because in general it is better to avoid working with huge keys. To split a bitmap across different keys instead of setting all the bits into a key, a trivial strategy is just to store M bits per key and obtain the key name with bit-number/M and the Nth bit to address inside the key with bit-number MOD M.

# HyperLogLogs
A HyperLogLog is a probabilistic data structure used in order to estimate the cardinality of a set. Usually counting unique items requires using an amount of memory proportional to the number of items you want to count, because you need to remember the elements you have already seen in the past in order to avoid counting them multiple times. However there is a set of algorithms that trade memory for precision: you end up with an estimated measure with a standard error, which in the case of the Redis implementation is less than 1%.