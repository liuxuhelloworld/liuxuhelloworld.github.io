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

# Netty Server
All Netty servers require the following:
- at least one **ChannelHandler**, this component implements the server's processing of data received from the client, its business logic
- bootstrapping, this is the startup code that configures the server. At a minimum, it binds the server to the port on which it will listen for connection requests

**ChannelHandler**, the parent of a family of interfaces whose implementations receive and react to event notifications. In Netty applications, all data-processing logic is contained in implementations of these core abstractions.