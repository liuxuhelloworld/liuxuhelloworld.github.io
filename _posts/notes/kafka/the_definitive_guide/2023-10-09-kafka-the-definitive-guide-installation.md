# Hardware Selection
### disk throughput
The performance of producer clients will be most directly influenced by the throughput of the broker disk that is used for storing log segments. Kafka messages must be committed to local storage when they are produced, and most clients will wait until at least one broker has confirmed that messages have been committed before considering the send successful. This means that faster disk writes will equal lower produce latency. 

The obvious decision when it comes to disk throughput is whether to use traditional spinning hard drives (HDD) or solid-state disks (SSD).

### disk capacity
The amount of disk capacity that is needed is determined by how many messages need to be retained at any time.

### memory
The normal mode of operation for a Kafka consumer is reading from the end of the partition, where the consumer is caught up and lagging behind the producers very little, if at all. In this situation, the messages the consumer is reading are optimally stored in the system's page cache, resulting in faster reads than if the broker has to reread the messages from disk. Therefore, having more memory avaiable to the system for page cache will improve the performance of consumer clients. This is the main reason it is not recommended to have Kafka collocated on a system with any other significant application, as they will have to share the use of the page cache. This will decrease the consumer performance for Kafka.

### networking
The available network throughput will specify the maximum amount of traffic that Kafka can handle. This is often the governing factor, combined with disk storage, for cluster sizing.

### CPU
Processing power is not as important as disk and memory, but it will affect overall performance of the broker to some extent. Ideally, clients should compress messages to optimize network and disk usage. The Kafka broker must decompress all message batches, however, in order to validate the checksum of the individual messages and assign offsets. It then needs to recompress the message batch in order to store it on disk. This is where the majority of Kafka's requirement for processing power comes from.

# Configuration
### broker.id
Every Kafka broker must have an integer identifier, which is set using the **broker.id** configuration. By default, this integer is set to 0, but it can be any value. The most important thing is that the integer must be unique within a single Kafka cluster.

### port
The example configuration file starts Kafka with a listener on TCP port 9092.

### num.partitions
Many users will have the partition count for a topic be equal to, or a multiple of, the number of brokers in the cluster. This allows the partitions to be evenly distributed to the brokers, which will evenly distribute the message load.

There are several factors to consider when choosing the number of partitions: What is the throughput you expect to achieve for the topic? What is the maximum throughput you expect to achieve when consuming from a single partition? You may want many partitions but not too many, as each partition uses memory and other resources on the broker and will increase the time for leader elections.

### log.segment.bytes
As messages are produced to the Kafka broker, they are appended to the current log segment for the partition. Once the log segment has reached the size specified by the **log.segment.bytes** parameter, which default to 1 GB, the log segment is closed and a new one is opened. Once a log segment has been closed, it can be considered for expiration.

### log.retention.ms
### log.retention.bytes
The log retention settings operate on log segments, not individual messages.

# Kafka Cluster
There are only two requirements in the broker configuration to allow multiple Kafka brokers to join a single cluster. The first is that all brokers must have the same configuration for the **zookeeper.connect** parameter. The second requirement is that all brokers in the cluster must have a unique value for the **broker.id** parameter.

The appropriate size for a Kafka cluster is determined by several factors. The first factor to consider is how much disk capacity is required for retaining messages and how much storage is available on a single broker. In addition, using replication will increase the storage requirements by at least 100%, depending on the replication factor chosen.