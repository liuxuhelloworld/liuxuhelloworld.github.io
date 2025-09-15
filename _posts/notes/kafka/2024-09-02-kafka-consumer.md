The Kafka consumer works by issuing fetch requests to the brokers leading the partitions it wants to consume. The consumer specifies its offset in the log with each request and receives back a chunk of log beginning from that position. The consumer thus has significant control over this position and can rewind it to re-consume data if need be.

## push vs. pull
Kafka follows a traditional design, shared by most messaging systems, where data is pushed to brokers from the producer and pulled from the broker by the consumer. 

A push-based system has difficulty dealing with diverse consumers as the broker controls the rate at which data is transferred. A pull-based system has the nice property that the consumer can just be fall behind and catch up when it can.

Another advantage of a pull-based system is that it lends itself to aggressive batching of data sent to the consumer. The consumer always pulls all available messages after its current position in the log (or up to some configurable max size). So one gets optimal batching without introducing unnecessary latency.

The deficiency of a naive pull-based system is that if the broker has no data the consumer may end up polling in a tight loop, effectively busy-waiting for data to arrive. To avoid this we have parameters in our pull request that allow the consumer request to block in a long pull waiting until data arrives (and optionally waiting until a given number of bytes is available to ensure large transfer size).

## consumer position
Keeping track of what has been consumed is, surprisingly, one of the key performance points of a messaging system. 

Most messaging systems keep metadata about what messages have been consumed on the broker. That is, as a message is handed out to a consumer, the broker either records the fact locally immediately or it may wait for acknowledgement from the consumer. This is a fairly intuitive choice.

What is perhaps not obvious is that getting the broker and consumer to come into agreement about what has been consumed is not a trivial problem. If the broker records a message as **consumed** immediately every time it is handed out over the network, then if the consumer fails to process the message, that message will be lost. To solve this problem, many messaging systems add an acknowledgement feature which means that messages are only marked as **sent** not **consumed** when they are sent; the broker waits for a specific acknowledgement from the consumer to record the message as **consumed**. This strategy fixes the problem of losing messages, but creates new problems. First of all, if the consumer processes the message but fails before it can send an acknowledgement then the message will be consumed twice. The second problem is about performance, now the broker must keep multiple states about every single message. Tricky problems must be dealt with, like what to do with messages that are sent but never acknowledged.

Kafka handles this differently. Our topic is divided into a set of totally ordered partitions, each of which is consumed by exactly one consumer within each subscribing consumer group at any given time. This means that the position of a consumer in each partition is just a single integer, the offset of the next message to consume. This makes the state about what has been consumed very small, just one number for each partition. This state can be periodically checkpointed. This makes the equivalent of message acknowledgements very cheap.

There is a side benefit of this decision. A consumer can deliberately rewind back to an old offset and re-consume data.