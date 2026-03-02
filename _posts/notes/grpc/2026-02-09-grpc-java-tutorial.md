# Why use gRPC?
With gRPC we can define our service once in a **.proto** file and generates clients and servers in any of gRPC's supported languages, which in turn can be run in environments ranging from servers inside a large data center to your own tablet, all the complexity of communication between different languages and environments is handled for you by gRPC. We also get all the advantages of working with protocol buffers, including efficient serialization, a simple IDL, and easy interface updating.

# Creating the server
There are two parts to making our gRPC service do its job:
- overriding the service base class generated from our service definition: doing the actual work of our service
- running a gRPC server to listen for requests from clients and return the service responses

# Creating the client
To call service methods, we first need to create a *stub*, or rather, two subs:
- a *blocking/synchronous* stub: this means that the RPC call waits for the server to respond, and will either return a response or raise an exception
- a *non-blocking/asynchronous* stub that makes non-blocking calls to the server, where the response is returned asynchronously. You can make certain types of streaming call only using the asynchronous stub

The generated class contains stubs for use by gRPC clients to call methods defined by the service. Each stub wraps a **Channel**, supplied by the user of the generated code. The stub uses this channel to send RPCs to the service.

gRPC Java generates code for three types of stubs: asynchronous, blocking, and future. Each type of stub has a corresponding class in the generated code, such as **ServiceNameStub**, **ServiceNameBlockingStub**, and **ServiceNameFutureStub**.

## asynchronous stub
RPCs made via an asynchronous stub operate entirely through callbacks on **StreamObserver**. 

The asynchronous stub contains one Java method for each method from the service definition.

## blocking stub
RPCs made through a blocking stub, as the name implies, block until the response from the service is available. 

The blocking stub contains one Java method for each unary and server-streaming method in the service definition. Blocking stubs do not support client-streaming or bidirectional-streaming RPCs.

## future stub
RPCs made via a future stub wrap the return value of the asynchronous stub in a **GrpcFuture\<ResponseType\>**, which implements the **com.google.common.util.concurrent.ListenableFuture** interface.

The future stub contains one Java method for each unary method in the service definition. Future stubs do not support streaming calls.