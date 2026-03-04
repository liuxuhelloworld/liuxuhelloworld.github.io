Netty is a Java framework for the rapid development of high-performance network applications. It encapsulates the complexities of network programming and makes the most recent advances in networking and web technologies accessible to a broader range of developers than ever before.

# Networking in Java
Developers who started out in the early days of networking spent a lot of time learning the intricacies of the C language socket libraries and dealing with their quirks on different operating systems. The earliest versions of Java (1995-2002) introduced enough of an object-oriented facade to hide some of the thornier details, but creating a complex client/server protocol still required a lot of boilerplate code (and a fair amount of peeking under the hood to get it all working smoothly).

Those first Java APIs (**java.net**) supported only the so-called blocking functions provided by the native system socket libraries.

![blocking I/O](/assets/images/netty_in_action/blocking-io.jpg)

The native socket libraries have long included non-blocking calls, which provide considerably more control over the utilization of network resources. Java support for non-blocking I/O was introduced in 2002, with the JDK 1.4 package **java.nio**.

The class **java.nio.channels.Selector** is the linchpin of Java's non-blocking I/O implementation. It uses the event notification API to indicate which, among a set of non-blocking sockets, are ready for I/O.

![non-blocking I/O](/assets/images/netty_in_action/non-blocking-io.jpeg)

Non-blocking I/O model provides much better resource management than the blocking I/O model:
- many connections can be handled with fewer threads, and thus with far less overhead due to memory management and context-switching
- threads can be retargeted to other tasks when there is no I/O to handle

Although many applications have been built using the Java NIO API directly, doing so correctly and safely is far from trivial. In particular, processing and dispatching I/O reliably and efficiently under heavy load is a cumbersome and error-prone task best left to a high-performance networking expert --- Netty.

# Introducing Netty
A fundamental concept of object orientation: hide the complexity of underlying implementations behind simpler abstractions. This principle has stimulated the development of numerous frameworks that encapsulate solutions to common programming tasks, many of them germane to distributed systems development.

In the networking domain, Netty is the preeminent framework for Java. Harnessing the power of Java's advanced APIs behind an easy-to-use API, Netty leaves you free to focus on what really interests you, the unique value of your application.

Asynchronous, that is, *un-synchronized*, events are certainly familiar. Consider email: you may or may not get a response to a message you have sent, or you may receive an unexpected message even while sending one. Asynchronous events can also have an *ordered* relationship. You generally get an answer to a question only after you have asked it, and you may be able to do something else while you are waiting for it.

In everyday life, asynchronous just happens, so you may not think about it much. But getting a computer program to work the same way presents some very special problems. In essence, a system that is both asynchronous and event-driven exhibits a particular and, to us, extremely valuable kind of behavior: it can respond to events occuring at any time and in any order. This capability is critical for achieving the highest levels of scalability, defined as "the ability of a system, network, or process to handle a growing amount of work in a capable manner or its ability to be enlarged to accommodate that growth."

What is the connection between asynchronous and scalability?
- non-blocking network calls free us from having to wait for the completion of an operation. Fully asynchronous I/O builds on this feature and carries it a step further: an asynchronous method returns immediately and notifies the user when it is complete, directly or at a later time
- selectors allow us to monitor many connections for events with many fewer threads

# Netty's Core Components
A **Channel** is a basic construct of Java NIO. It represents an open connection to an entity such as a hardware device, a file, a network socket, or a program component that is capable of performing one or more distinct I/O operations, for example reading or writing.

A *callback* is simply a method, a reference to which has been provided to another method. This enables the latter to call the former at an appropriate time. Callbacks are used in a broad range of programming situations and represent one of the most common ways to notify an interested party that an operation has completed. Netty uses callbacks internally when handling events; when a callback is triggered the event can be handled by an implementation of **interface ChannelHandler**.

Netty provides an extensive set of predefined handlers that you can use out of the box, including handlers for protocols such as HTTP and SSL/TLS. Internally, **ChannelHandler**s use events and futures themselves, making them consumers of the same abstractions you applications will employ.

A **Future** provides another way to notify an application when an operation has completed. This object acts as a placeholder for the result of an asynchronous operation; it will complete at some point in the future and provide access to the result.

The JDK ships with **interface java.util.concurrent.Future**, but the provided implementations allow you only to check manually whether the operation has completed or to block until it does. This is quite cumbersome, so Netty provides its own implementation, **ChannelFuture**, for use when an asynchronous operation is executed.

**ChannelFuture** provides additional methods that allow us to register one or more **ChannelFutureListener** instances. The listenr's callback method, **operationComplete()**, is called when the operation has completed. The listener can then determine whether the operation completed successfully or with an error. If the latter, we can retrieve the **Throwable** that was produced. In short, the notification mechanism provided by the **ChannelFutureListener** eliminates the need for manually checking operation completion.

You can think of **ChannelFutureListener** as a more elaborate version of a callback. In fact, callbacks and **Future**s are complementary mechanisms; in combination they make up one of the key building blocks of Netty itself.

Netty's asynchronous programming model is built on the concepts of **Future**s and callbacks, with the dispatching of events to handler methods happening at a deeper level. Taken together, these elements provide a processing environment that allows the logic of your application to evole independently of any concerns with network operations. This is a key goal of Netty's design approach.

Netty abstracts the **Selector** away from the application by firing events, eliminating all the handwritten dispatch code that would otherwise be required.