In gRPC, a client application can directly call a method on a server application on a different machine as if it were a local object, making it easier for you to create distributed applications and services. As in many RPC systems, gRPC is based around the idea of defining a service, specifying the methods that can be called remotely with their parameters and return types. On the server side, the server implements this interface and runs a gRPC server to handle client calls. On the client side, the client has a stub that provides the same methods as the server.

![gRPC illustration](/assets/images/grpc/grpc-illustration.jpg)

gRPC clients and servers can run and talk to each other in a variety of environments and can be written in any of gRPC's supported languages. So, for example, you can easily create a gRPC server in java with clients in Go, Python, or Ruby.

By default, gRPC uses **Protocol Buffers**, Google's mature open source mechanism for serializing structured data (although it can be used with other data formats such as JSON).

The first step when working with protocol buffers is to define the structure for the data you want to serialize in a *proto file*: this is an ordinary text file with a **.proto** extension. Protocol buffer data is structured as *messages*, where each message is a small logical record of information containing a series of name-value pairs called *fileds*. Then, once you've specified your data structures, you use the protocol buffer compiler **protoc** to generate data access classes in your preferred languages from your proto definition. These provide simple accessors for each field as well as methods to serialize/parse the whole structure to/from raw bytes.

You define gRPC services in ordinary proto files, with RPC method parameters and return types specified as protocol buffer messages.

gRPC uses **protoc** with a special gRPC plugin to generate code from your proto file: you get generated gRPC client and server code, as well as the regular protocol buffer code for populating, serializing, and retrieving your message types.

# gRPC Service Types
gRPC lets you define four kinds of service methods:
- Unary RPCs where the client sends a single request to the server and gets a single response back, just like a normal function call
- Server streaming RPCs where the client sends a request to the server and gets a stream to read a sequence of messages back. The client reads from the returned stream until there are no more messages. gRPC guarantees message ordering within an individual RPC call
- Client streaming RPCs where the client writes a sequence of messages and sends them to the server, again using a provided stream. Once the client has finished writing the messages, it waits for the server to read them and return its response. Again gRPC guarantees message ordering within an individual RPC call
- Bidirectional streaming RPCs where both sides send a sequence of messages using a read-write stream. The two streams operate independently, so clients and servers can read and write in whatever order they like. The order of messages in each stream is preserved

# RPC Lift Cycle
For unary RPC:
1. Once the client calls a stub method, the server is notified that the RPC has been invoked with the client's metadata for this call, the method name, and the specified deadline if applicable.
2. The server can then either send back its own initial metadata (which must be sent before any response) straight away, or wait for the client's request message. Which happens first, is application-specific.
3. Once the server has the client's request message, it does whatever work is necessary to create and populate a response. The response is then returned to the client together with status details (status code and optional status message) and optional trailing metadata.
4. If the response is OK, then the client gets the response, which completes the call on the client side.

A server-streaming RPC is similar to a unary RPC, except that the server returns a stream of messages in response to a client's request. After sending all its messages, the server's status details (status code and optional status message) and optional trailing metadata are sent to the client. This completes processing on the server side. The client completes once it has all the server's messages.

A client-streaming RPC is similar to a unary RPC, except that the client sends a stream of messages to the server instead of a single message. The server responds with a single message (along with its status details and optional trailing metadata), typically but not necessarily after it has received all the clien't messages.

In a bidirectional streaming RPC, the call is initialized by the client invoking the method and the server receiving the client metadata, method name, and deadline. The server can choose to send back its initial metadata or wait for the client to start streaming messages. Client- and server-side stream processing is application specific. Since the two streams are independent, the client and server can read and write messages in any order.

gRPC allows clients to specify how long they are willing to wait for an RPC to complete before the RPC is terminated with a **DEADLINE_EXCEEDED** error. On the server side, the server can query to see if a particular RPC has timed out, or how much time is left to complete the RPC.

In gRPC, both the client and server make independent and local determinations of the success of the call, and their conclusions may not match. This means that, for example, you could have an RPC that finishes successfully on the server side ("I have sent all my responses") but fails on the client side ("The responses arrived after my deadline). It's also possible for a server to decide to complete before a client has sent all its requests.

Either the client or the server can cancel an RPC at any time. A cancellation terminates the RPC immediately so that no further work is done. Changes made before a cancellation are not rolled back.

# Metadata
Metadata is information about a particular RPC call in the form of a list of key-value pairs, where the keys are strings and values are typically strings, but can be binary data.

Keys are case insensitive and consist of ASCII letters, digits, and special characters **-**, **_**, **.** and must not start with **grpc-** (which is reserved for gRPC itself). Binary-valued keys end in **-bin** while ASCII-valued keys do not.

User-defined metadata is not used by gRPC, which allows the client to provide information associated with the call to the server and vice versa.

# Channels
A gRPC channel provides a connection to a gRPC server on a specified host and port. It is used when creating a client stub. Clients can specify channel arguments to modify gRPC's default behavior. A channel has state, including **connected** and **idle**.

# gRPC vs binary blob over HTTP/2
This is largely what gRPC is on the wire. However gRPC is also a set of libraries that will provide higher-level features consistently across platforms that common HTTP libraries typically do not. Examples of such features include:
- interaction with flow-control at the application layer
- cascading call-cancellation
- load balancing & failover

# gRPC vs REST
gRPC largely follows HTTP semantics over HTTP/2 but we explicitly allow for full-duplex streaming. We diverge from typical REST conventions as we use static paths for performance reasons during call dispatch as parsing call parameters from paths, query parameters and payload body adds latency and complexity. We have also formalized a set of errors that we believe are more directly applicable to API use cases than the HTTP status codes.