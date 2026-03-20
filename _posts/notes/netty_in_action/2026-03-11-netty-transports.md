The data that flows through a network always has the same type: bytes. How these bytes are moved around depends mostly on what we refer to as the network transport, a concept that helps us to abstract away the underlying mechanics of data transfer.

If you have experience with network programming in Java, you may have discovered at some point that you needed to support a great many more concurrent connections than expected. If you then tried to switch from a blocking to a non-blocking transport, you might have encountered problems because the two network APIs are quite different.

Netty, however, layers a common API over all its transport implementations, making such a conversion far simpler than you can achieve using the JDK directly. We'll study this common API, constrasting it with JDK to demonstrate its far greater ease of use.

Keep in mind that the broad range of functionality offered by Netty relies on a small number of interfaces. This means that you can make significant modifications to application logic without wholescale refactoring of you code base.

# Transports
Netty comes bundled with several transports that are ready for use. Because not all of them support every protocol, you have to select a transport that is compatible with the protocols employed by your application.
- NIO, uses the **java.nio.channels** package as a foundation, a selector-based approach
- Epoll, uses JNI for epoll() and non-blocking IO. This transport supports features available only on Linux and is faster than the NIO transport as well as fully non-blocking
- OIO, uses the **java.net** package as a foundation, uses blocking streams
- Local, a local transport that can be used to communicate in the VM via pipes
- Embedded, an embedded transport, which allows using **ChannelHandler**s without a true network-based transport. This can be quite useful for testing your **ChannelHandler** implementations

## NIO
NIO provides a fully asynchronous implementation of all I/O operations. It makes use of the selector-based API that has been available since the NIO subsystem was introduced in JDK 1.4.

The basic concept behind the selector is to serve as a registry where you request to be notified when the state of a **Chanenel** changes. After the application reacts to the change of state, the selector is reset and the process repeats, running on a thread that checks for changes and responds to them accordingly.

![Netty NIO](/assets/images/netty_in_action/netty-nio.jpg)

## Epoll
Netty's NIO transport is based on the common abstraction for asynchronous/non-blocking networking provided by Java. Although this ensures that Netty's non-blocking API will be usable on any platform, it also entails limitations, because the JDK has to make compromises in order to deliver the same capabilities on all systems.

The growing importance of Linux as a platform for high-performance networking has led to the development of a number of advanced features, including *epoll*, a highly scalable I/O event-notification feature. This API, avaiable since version 2.5.44 (2002) of the Linux kernel, provides better performance than the older POSIX **select** and **poll** systems calls and is now the de facto standard for non-blocking networking on Linux.

Netty provides an NIO API for Linux that uses epoll in a way that's more consistent with its own design and less costly in the way it uses interrupts. Consider utilizing this version if your applications are intended for Linux; you will find that performance under heavy load is superior to that of the JDK's NIO implementation.

## OIO
Netty's OIO transport implementation represents a compromise: it is accessed via the common transport API, but because it's built on the blocking implementation of **java.net**, it's not asynchronous. 

OIO is well-suited to certain uses. For example, you might need to port legacy code that uses libraries that make blocking calls (such as JDBC) and it may not be practical to convert the logic to non-blocking. Instead, you could use Netty's OIO transport in the short term, and port your code later to one of the pure asynchronous transports.

In the **java.net** API, you usually have one thread that accepts new connections arriving at the listening **ServerSocket**. A new socket is created for the interaction with the peer, and a new thread is allocated to handle the traffic. This is required because any I/O operation on a specific socket can block at any time. Handling multiple sockets with a single thread can easily lead to a blocking operation on one socket tying up all the others as well.

![Netty OIO](/assets/images/netty_in_action/netty-oio.jpg)

## Local 
Netty provides a local transport for asynchronous communication between clients and servers running in the same JVM. In this transport, the **SocketAddress** associated with a server **Channel** isn't bound to a physical network address; rather, it's stored in a registry for as long as the server is running and is deregistered when the **Channel** is closed. Because the transport doesn't accept real network traffic, it can't interoperate with other transport implementations. Therefore, a client wishing to connect to a server (in the same JVM) that uses this transport must also use it.