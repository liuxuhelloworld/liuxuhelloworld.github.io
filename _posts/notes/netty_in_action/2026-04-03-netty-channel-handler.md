## Channel states
- ChannelUnregistered, the channel was created, but isn't registered to an **EventLoop**
- ChannelRegistered, the channel is registered to an **EventLoop**
- ChannelActive, the channel is active (connected to its remote peer). It is now possible to receive and send data
- ChannelInactive, the channel isn't connected to the remote peer

![Channel state model](/assets/images/netty_in_action/channel-state-model.jpg)

As these state changes occur, corresponding events are generated. These are forwarded to **ChannelHandler**s in the **ChannelPipeline**, which can then act on them.

## ChannelHandler
- handlerAdd, called when a **ChannelHandler** is added to a **ChannelPipeline**
- handlerRemoved, called when a **ChannelHandler** is removed from a **ChannelPipeline**
- exceptionCaught, called if an error occurs in the **ChannelPipeline** during processing

## ChannelInboundHandler
- channelRegistered, invoked when a **Channel** is registered to its **EventLoop** and is able to handle I/O
- channelUnregistered, invoked when a **Channel** is deregistered from its **EventLoop** and can't handle any I/O
- channelActive, invoked when a **Channel** is active; the **Channel** is connected/bound and ready
- channelInactive, invoked when a **Channel** leaves active state and is no longer connected to its remote peer
- channelRead, invoked if data is read from the **Channel**
- channelReadComplete, invoked when a read operation on the **Channel** has completed
- channelWritabilityChanged, invoked when the writability state of the **Channel** changes

## ChannelOutboundHandler
- bind, invoked on request to bind the **Channel** to a local address
- connect, invoked on request to connect the **Channel** to the remote peer
- disconnect, invoked on request to disconnect the **Channel** from the remote peer
- close, invoked on request to close the **Channel**
- deregister, invoked on request to deregister the **Channel** from its **EventLoop**
- read, invoked on request to read more data from the **Channel**
- flush, invoked on request to flush queued data to the remote peer through the **Channel**
- write, invoked on request to write data through the **Channel** to remote peer

## ChannelHandlerAdapter
You can use the classes **ChannelInboundHandlerAdapter** and **ChannelOutboundHandlerAdapter** as starting point for your own **ChannelHandler**s. These adapters provide basic implementations of **ChannelInboundHandler** and **ChannelOutboundHandler** respectively. The method bodies provided in **ChannelInboundHandlerAdapter** and **ChannelOutboundHandlerAdapter** call the equivalent methods on the associated **ChannelHandlerContext**, thereby forwarding events to the next **ChannelHandler** in the pipeline.

## memory leaks
Whenever you act on data by calling **ChannelInboundHandler.channelRead()** or **ChannelOutboundHandler.write()**, you need to ensure that there are no resource leaks. Netty uses reference counting to handle pooled **ByteBuf**s, so it's important to adjust the reference count after you have finished using a **ByteBuf**.

To assist you in diagnosing potential problems, Netty provides class **ResourceLeakDetector**, which will sample about 1% of your application's buffer allocations to check for memory leaks. The overhead involved is very small.

Because consuming inbound data and releasing it is such a common task, Netty provides a special **channelInboundHandler** implementation called **SimpleChannelInboundHandler**. This implementation will automatically release a message once it's consumed by **channelRead0()**.

As for the **ChannelOutboundHandler**, it is the responsibility of the user to call **ReferenceCountUtil.release()** if a message is consumed or discarded and not passed to the next **ChannelOutboundHandler** in the **ChannelPipeline**. If the message reaches the actual transport layer, it will be released automatically when it's written or the **Channel** is closed.

## ChannelPipeline
Every new **Channel** that's created is assigned a new **ChannelPipeline**. This association is permanent; the **Channel** can neither attach another **ChannelPipeline** nor detach the current one.

If you think of a **ChannelPipeline** as a chain of **ChannelHandler** instances that intercept the inbound and outbound events that flow through a **Channel**, it's easy to see how the interaction of these **ChannelHandler**s can make up the core of an application's data and event-processing logic.

![ChannelPipeline](/assets/images/netty_in_action/channel-pipeline-illustration.jpg)

As the pipeline propagates an event, it determines whether the type of the next **ChannelHandler** in the pipeline matches the direction of movement. If not, the **ChannelPipeline** skips that **ChannelHandler** and proceeds to the next one, until it finds one that matches the desired direction.

A **ChannelHandler** can modify the layout of a **ChannelPipeline** in real time by adding, removing, or replacing other **ChannelHandler**s.

## ChannelHandlerContext
A **ChannelHandlerContext** represents an association between a **ChannelHandler** and a **ChannelPipeline** and is created whenever a **ChannelHandler** is added to a **ChannelPipeline**. The primary function of a **ChanenelHandlerContext** is to manage the interaction of its associated **ChannelHandler** with others in the same **ChannelPipeline**.

![ChannelHandlerContext](/assets/images/netty_in_action/channel-handler-context.jpg)

The **ChannelHandlerContext** associated with a **ChannelHandler** never changes, so it's safe to cache a reference to it.

**ChannelHandlerContext** methods involve a shorter event flow than do the identically named methods available on other classes. This should be exploited where possible to provide maximum performance. To invoke processing starting with a specific **ChannelHandler**, you must refer to the **ChannelHandlerContext** that's associated with the **ChannelHandler** before that one. This **ChannelHandlerContext** will invoke the **ChannelHandler** that follows the one with which it's associated.

![event flow triggered by Channel](/assets/images/netty_in_action/event-flow-triggered-by-channel.jpg)

![event flow triggered by ChannelHandlerContext](/assets/images/netty_in_action/event-flow-triggered-by-channel-handler-context.jpg)

## exception handling
If an exception is thrown during processing of an inbound event, it will start to flow through the **ChannelPipeline** starting at the point in the **ChannelInboundHandler** where it was triggered. If an exception reaches the end of the pipeline, it's logged as unhandled. To define custom handling, you override **exceptionCaught()**. It's then your decision whether to propagate the exception beyond that point.

The options for handling normal completion and exceptions in outbound operations are based on the following notification mechanisms:
- every outbound operation returns a **ChannelFuture**. The **ChannelFutureListener**s registered with a **ChannelFuture** are notified of success or error when the operation completes
- almost all methods of **ChannelOutboundHandler** are passed an instance of **ChannelPromise**. As a subclass of **ChannelFuture**, **ChannelPromise** can also be assigned listeners for asynchronous notification