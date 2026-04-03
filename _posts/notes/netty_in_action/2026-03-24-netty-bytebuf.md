Netty's **ByteBuf** is an alternative to Java's **ByteBuffer**, a power implementation that addressses the limitations of the JDK API and provides a better API for network application developers.

Because all network communications involve the movement of sequences of bytes, an efficient and easy-to-use data structure is an obvious necessity. Netty's **ByteBuf** implementation meets and exceeds these requirements.

**ByteBuf** maintains two distinct indices: one for reading and one for writing. When you read from a **ByteBuf**, its **readerIndex** is incremented by the number of bytes read. Similarly, when you write to a **ByteBuf**, its **writerIndex** is incremented.

![ByteBuf example](/assets/images/netty_in_action/bytebuf-example.jpg)

# ByteBuf Usage Patterns
## heap buffers
The most frequently used **ByteBuf** pattern stores the data in the heap space of the JVM. Referred to as a *backing array*, this pattern provides fast allocation and deallocation in situations where pooling isn't in use.

## direct buffers
*Direct buffer* is another **ByteBuf** pattern. We expect that memory allocated for object creation will always come from the heap, but it doesn't have to, the **ByteBuffer** class that was introduced in JDK 1.4 with NIO allows a JVM implementation to allocate memory via native calls. This aims to avoid copying the buffer's contents to (or from) an intermediate buffer before (or after) each invocation of a native I/O operation.

The Javadoc for **ByteBuffer** states explicitly, "The contents of direct buffers will reside outside of the normal garbage-collected heap." This explains why direct buffers are ideal for network data transfer. If you data were contained in a heap-allocated buffer, the JVM would, in fact, copy your buffer to a direct buffer internally before sending it through the socket.

## composite buffers
The third and final pattern uses a *composite buffer*, which presents an aggregated view of multiple **ByteBuf**s. Netty implements this pattern with a subclass of **ByteBuf**, **CompositeByteBuf**, which provides a virtual representation of multiple buffers as a single, merged buffer.

To illustrate, let's consider a message composed of two parts, header and body, to be transmitted via HTTP. The two parts are produced by differnet application modules and assembled when the message is sent out. The application has the option of reusing the same message body for multiple messages. When this happens, a new header is created for each message. Because we don't want to reallocate both buffers for each message, **CompositeByteBuf** is a perfect fit; it eliminates unnecessary copying while exposing the common **ByteBuf** API.

# ByteBuf Segmentation
![ByteBuf segmentation](/assets/images/netty_in_action/bytebuf-segmentation.jpg)

The segment labeled discardable bytes contains bytes that have already been read. They can be discarded and the space reclaimed by calling **discardReadBytes()**. The initial size of this segment, stored in **readerIndex**, is 0, increasing as **read** operations are executed.

![ByteBuf after discarding read bytes](/assets/images/netty_in_action/bytebuf-after-discarding-read-bytes.jpg)

While you may be tempted to call **discardReadBytes()** frequently to maximize the writable segment, please be aware that this will most likely case memory copying because the readable bytes have to be moved to the start of the buffer. We avoid doing this only when it's really needed; for example, when memory is at a premium.

# ByteBufHolder
We often find that we need to store a variety of property values in addition to the actual data payload. An HTTP response is a good example; along with the content represented as bytes, there are status code, cookies, and so on. 

Netty provides **ByteBufHolder** to handle this common use case. **ByteBufHolder** also provides support for advanced features of Netty such as buffer pooling, where a **ByteBuf** can be borrowed from a pool and also be released automatically if required. 

**ByteBufHolder** is a good choice if you want to implement a message object that stores its payload in a **ByteBuf**.

# ByteBufAllocator
To reduce the overhead of allocating and deallocating memory, Netty implements pooling with the interface **ByteBufAllocator**. Netty provides two implementations of **ByteBufAllocator**: **PooledByteBufAllocator** and **UnpooledByteBufAllocator**. The former pools **ByteBuf** instances to improve performance and minimize memory fragmentation. This implementation uses an efficient approach to memory allocation known as *jemalloc* that has been adopted by a number of modern OSes. The latter implementation doesn't pool **ByteBuf** instances and returns a new instance every time it's called.