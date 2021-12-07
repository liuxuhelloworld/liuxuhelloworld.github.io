# Persistence
Kafka relies heavily on the filesystem for storing and caching messages.

Rather than maintain as much as possible in-memory and flush it all out to the filesystem in a panic when we run out of space, we invert that. All data is immediately written to a persistent log on the filesystem without necessarily flushing to disk. In effect this just means that it is transfered into the kernel's pagecache.

# Efficiency
Once poor disk access patterns have been eliminated, there are two common causes of inefficiency in this type of system: too many small I/O operations, and excessive byte copying.

The small I/O problem happens both between the client and the server and in the server's own persistent operations. To avoid this, our protocol is built around a "message set" abstraction that naturally groups messages together. This allows network requests to group messages together and amortize the overhead of the network roundtrip rather than sending a single message at a time. The server in turn appends chunks of messages to its log in one go, and the consumer fetches large linear chunks at a time. This simple optimization produces orders of magnitude speed up. Batching leads to larger network packets, larger sequential disk operations, contiguous memory blocks, and so on, all of which allows Kafka to turn a bursty stream of random messages writes into linear writes that flow to the consumers.

The other inefficiency is in byte copying. Modern unix operating systems offer a highly optimized code path for transferring data out of pagecache to a socket; in Linux this is done with the sendfile system call.

To understand the impact of sendfile, it is important to understand the common data path for transfer of data from file to socket:
1. the operating system reads data from the disk into pagecache in kernel space
2. the application reads the data from kernel space into a user-space buffer
3. the application writes the data back into kernel space into a socket buffer
4. the operating system copies the data from the socket buffer to the NIC buffer where it is sent over the network

Using sendfile, this re-copying is avoided by allowing the OS to send the data from pagecache to the network directly.

Using the zero-copy optimization, data is copied into pagecache exactly once and reused on each consumption instead of being stored in memory and copied out to user-space every time it is read. This combination of pagecache and sendfile means that on a Kafka cluster where the consumers are mostly caught up you will see no read activity on the disks whatsoever as they will be serving data entirely from cache.

In some cases the bottleneck is actually not CPU or disk but network bandwidth. Efficient compression requires compressing multiple messages together rather than compressing each message individually. Kafka supports this with an efficient batching format. A batch of messages can be clumped together compressed and sent to the server in this form. This batch of messages will be written in compressed form and will remain compressed in the log and will only be decompressed by the consumer.