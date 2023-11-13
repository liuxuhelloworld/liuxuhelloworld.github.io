Kafka consumers are typically part of a consumer group. When multiple consumers are subscribed to a topic and belong to the same consumer group, each consumer in the group will receive messages from a different subset of the partitions in the topic.

The main way we scale data consumption from a Kafka topic is by adding more consumers to a consumer group. It is common for Kafka consumers to do high-latency operations such as write to a database or a time-consuming computation on the data. In these cases, a single consumer can't possibly keep up with the rate data flows into a topic, and adding more consumers that share the load by having each consumer own just a subset of the partitions and messages is our main method of scaling. This is a good reason to create topics with a large number of partitions, it allows adding more consumers when the load increases. Keep in mind that there is no point in adding more consumers than you have partitions in a topic, some of the consumers will just be idle.

In addition to adding consumers in order to scale a single application, it is very common to have multiple applications that need to read data from the same topic. To make sure an application gets all the messages in a topic, ensure the application has its own consumer group. Unlike many traditional messaging systems, Kafka scales to a large number of consumers and consumer groups without reducing performance.

Moving partitions ownership from one consumer to another is called a *rebalance*. Rebalances are important because they provide the consumer group with high availability and scalability (allowing us to easily and safely add and remove consumers), but in the normal course of events they are fairly undesirable. During a rebalance, consumers can't consume messages, so a rebalance is basically a short window of unavailability of the entire consumer group. In addition, when partitions are moved from one consumer to another, the consumer loses its current state, if it was caching any data, it will need to refresh its caches, slowing down the application until the consumer sets up its state again.

The way consumers maintain membership in a consumer group and ownership of the partitions assigned to them is by sending heartbeats to a Kafka broker designated as the group coordinator.

# Configuration
### fetch.min.bytes
### fetch.max.wait.ms
By setting **fetch.min.bytes**, you tell Kafka to wait until it has enough data to send before responding to the consumer. **fetch.max.wait.ms** lets you control how long to wait. By default, Kafka will wait up to 500 ms. This results in up to 500 ms of extra latency in case there is not enough data flowing to the Kafka topic to satisfy the minimum amount of data to return.

### session.timeout.ms
### heartbeat.interval.ms
The amount of time a consumer can be out of contact with the brokers while still considered alive defaults to 3 seconds. If more than **session.timeout.ms** passes without the consumer sending a heartbeat to the group coordinator, it is considered dead and the group coordinator will trigger a rebalance of the consumer group to allocate partitions from the dead consumer to the other consumers in the group. **heartbeat.interval.ms** must be lower than **session.timeout.ms**, and is usually set to one-third of the timeout value.

# Commits and Offsets
We call the action of updating the current position in the partition a **commit**. 

How does a consumer commit an offset? It produces a message to Kafka, to a special **__consumer_offsets** toipic, with the committed offset for each partition.

The easiest way to commit offsets is to allow the consumer to do it for you. If you configure **enable.auto.commit=true**, then every five seconds the consumer will commit the largest offset you client received from **poll()**. It is possible to configure the commit interval to commit more frequently and reduce the window in which records will be duplicated, but it is impossible to completely eliminate them. With autocommit enabled, a call to poll will always commit the last offset returned by the previous poll. It doesn't know which events were actually processed, so it is critical to always process all the events returned by **poll()** before calling **poll()** again.

By setting **auto.commit.offset=false**, offsets will only be committed when the application explicitly chooses to do so. The simplest and most reliable of the commit API is **commitSync()**. This API will commit the latest offset returned by **poll()** and return once the offset is committed, throwing an exception if commit fails for some reason.

One drawback of manual commit is that the application is blocked until the broker responds to the commit request. This will limit the throughput of the application. Throughput can be improved by committing less frequently, but then we are increasing the number of potential duplicates that a rebalance will create. Another option is the asynchronous commit API. Instead of waiting for the broker to respond to a commit, we just send the request and continue on.