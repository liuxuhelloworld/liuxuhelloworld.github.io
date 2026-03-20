We will explore Netty from two distinct but closely related points of view: as a class library and as a framework. Both are essential to writing efficient, reusable, and maintainable code with Netty.

From a high-level perspective, Netty addresses two corresponding areas of concern, which we might label broadly as *technical* and *architectural*. First, its asynchronous and event-driven implementation, built on Java NIO, guarantees maximum application performance and scalability under heavy load. Second, Netty embodies a set of design patterns that decouple application logic from the network layer, simplying development while maximizing the testability, modularity, and reusability of code.

# Channel, EventLoop, and ChannelFuture
**Channel**, **EventLoop**, and **ChannelFuture** can be thought of as representing Netty's networking abstraction:
- Channel: sockets
- EventLoop: control flow, multithreading, concurrency
- ChannelFuture: asynchronous notification

Basic I/O operations (**bind()**, **connect()**, **read()**, and **write()**) depend on primitives supplied by the underlying network transport. In Java-based networking, the fundamental construct is **class Socket**. Netty's **Channel** interface provides an API that greatly reduces the complexity of working directly with **Socket**s. Additionally, **Channel** is the root of an extensive class hierarchy having many predefined, specialized implementations, for example, **NioDatagramChannel**, **NioSocketChannel**.

The **EventLoop** defines Netty's core abstraction for handling events that occur during the lifetime of a connection.
![EventLoop](/assets/images/netty_in_action/EventLoop.jpeg)

- an **EventLoopGroup** contains one or more **EventLoop**s
- an **EventLoop** is bound to a single **Thread** for its lifetime
- all I/O events processed by an **EventLoop** are handled on its dedicated **Thread**
- a **Channel** is registered for its lifetime with a single **EventLoop**
- a single **EventLoop** may be assigned to one or more **Channel**s

Note that this design, in which the I/O for a given **Channel** is executed by the same **Thread**, virtually eliminates the need for synchronization.

All I/O operations in Netty are asynchronous. Because an operation may not return immediately, we need a way to determine its result at a later time. For this purpose, Netty provides **ChannelFuture**, whose **addListener()** method registers a **ChannelFutureListener** to be notified when an operation has completed (whether or not successfully). Think of a **ChannelFuture** as a placeholder for the result of operation that's to be executed in the future. When exactly it will be executed may depend on several factors and thus be impossible to predict with precision, but it is certain that it will be executed. Furthermore, all operations belonging to the same **Channel** are guaranteed to be executed in the same order in which they were invoked.

# ChannelHandler, ChannelPipeline and ChannelHandlerAdapter
From the application developer's standpoint, the primary component of Netty is the **ChannelHandler**, which serves as the container for all application logic that applies to handling inbound and outbound data. As an example, **ChannelInboundHandler** is a subinterface you'll implement frequently. This type receives inbound events and data to be handled by your application's business logic. You can also flush data from a **ChannelInboundHandler** when you're sending a response to a connected client. The business logic of your application will often reside in one or more **ChannelInboundHandler**s.

A **ChannelPipeline** provides a container for a chain of **ChannelHandler**s and defines an API for propagating the flow of inbound and outbound events along the chain. When a **Channel** is created, it is automatically assigned its own **ChannelPipeline**.

**ChannelHandler** has been designed specifically to support a broad range of uses, and you can think of it as a generic container for any code that processes events (including data) coming and going through the **ChannelPipeline**. The movement of any event through the pipeline is the work of the **ChannelHandler**s that have been installed during the initialization, or bootstrapping phase of the application. These objects receive events, execute the processing logic for which they have been implemented, and pass the data the next handler in the chain. The order in which they are executed is determined by the order in which they were added. For all practical purposes, it's this ordered arrangement of **ChannelHandler**s that we refer to as the **ChannelPipeline**. 

![ChannelHandler and ChannelPipeline](/assets/images/netty_in_action/channel-handler-and-channel-pipeline.jpg)

If a message or any other inbound event is read, it will start from the head of the pipeline and be passed to the first **ChannelInboundHandler**. This handler may or may not actually modify the data, depending on its specific function, after which the data will be passed to the next **ChannelInboundHandler** in the chain. Finally, the data will reach the tail of the pipeline, at which point all processing is terminated. The outbound movement of data (that is, data being written) is identical in concept. In this case, data flows from the tail through the chain of **ChannelOutboundHandler**s until it reaches the head. Beyond this point, outbound data will reach the network transport, shown here as a **Socket**. Typically, this will trigger a write operation.

When a **ChannelHandler** is added to a **ChannelPipeline**, it's assigned a **ChannelHandlerContext**, which represents the binding between a **ChannelHandler** and the **ChannelPipeline**. Although this object can be used to obtain the underlying **Channel**, it's mostly utilized to write outbound data.

Netty provides a number of default handler implementations in the form of adapter classes, which are intended to simplify the development of an application's processing logic. For example, **ChannelHandlerAdapter**, **ChannelInboundHandlerAdapter**, and **ChannelOutboundHandlerAdapter**. These adapter classes reduce the effort of writing custom **ChannelHandler**s to a bare minimum, because they provide default implementations of all the methods defined in the corresponding interface.

When you send or receive a message with Netty, a data conversion takes place. An inbound message will be *decoded*; that is, converted from bytes to another format, typically a Java object. If the message is outbound, the reverse will happen: it will be *encoded* to bytes from its current format. The reason for both conversions is simple: network data is always a series of bytes. Just as there are adapter classes to simplify the creation of channel handlers, all of the encoder/decoder adapter classes provided by Netty implement either **ChannelInboundHandler** or **ChannelOutboundHandler**.

Most frequently you application will employ a handler that receives a decoded message and applies business logic to the data. To create such a **ChannelHandler**, you need only extend the base class **SimpleChannelInboundHandler\<T\>**, where **T** is the Java type of the message you want to process.

# Bootstrapping
Netty's bootstrap classes provide containers for the configuration of an application's network layer, which involves either binding a process to a given port or connecting one process to another one running on a specified host at a specified port. In general, we refer to the former use case as bootstrapping a server and the latter as bootstrapping a client. Accordingly, there are two types of bootstraps: one intended for clients (called simply **Bootstrap**), and the other for servers (**ServerBootstrap**).

Bootstrapping a client requires only a single **EventLoopGroup**, but a **ServerBootstrap** requires two (which can be the same instance). A server needs two distinct set of **Channel**s. The first set will contain a single **ServerChannel** representing the server's own listening socket, bound to a local port. The second set will contain all of the **Channel**s that have been created to handle incoming client connections, one for each connection the server has accepted.

![Server need two EventLoopGroups](/assets/images/netty_in_action/server-need-two-event-loop-groups.jpg)

# Netty Server
All Netty servers require the following:
- at least one **ChannelHandler**, this component implements the server's processing of data received from the client, its business logic
- bootstrapping, this is the startup code that configures the server. At a minimum, it binds the server to the port on which it will listen for connection requests

**ChannelHandler**, the parent of a family of interfaces whose implementations receive and react to event notifications. In Netty applications, all data-processing logic is contained in implementations of these core abstractions.