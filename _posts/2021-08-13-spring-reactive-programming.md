# Imperative Programming vs Reactive Programming
Imperative programming is simple: you write code as a list of instructions to be followed, one at a time, in the order that they're encountered. A task is performed and the program waits for it to complete before moving to the next task. At each step along the way, the data that's to be processed must be fully available so that it can be processed as a whole. While a task is being performed, and especially if it's an I/O task such as writing data to a database or fetching data from a remote server, the thread that invoked that task is blocked, unable to do anything else until the task completes. Many programming languages, including Java, support concurrent programming. But managing concurrency in multiple threads is challenging. More threads mean more complexity.

Rather than describe a set of steps that are to be performed sequentially, reactive programming involves describing a pipeline or stream through which data flows. Rather than requiring the data be available to be processed as a whole, a reactive stream processes data as it becomes available.

# Reactive Streams
Reactive Streams aims to provide a standard for asynchronous stream processing with non-blocking backpressure. 

Backpressure is a means by which consumers of data can avoid being overwhelmed by an overly fast data source, by establishing limits on how much they're willing to handle.

The Reactive Streams specification can be summed up by four interface definitions: **Publisher**, **Subscriber**, **Subscription**, and **Processor**. 

A **Publisher** produces data that it sends to a **Subscriber** per a **Subscription**. 
```java
public interface Publisher<T> {
		void subscribe(Subscriber<? super T> subscriber);
}
```

Once a **Subscriber** has subscribed, it can receive events from the **Publisher**. Those events are sent via methods on the **Subscriber** interface.
```java
public interface Subscriber<T> {
		void onSubscribe(Subscription sub);
		void onNext(T item);
		void onError(Throwable ex);
		void onComplete();
}
```

When the **Publisher** calls **onSubscribe()**, it passes a **Subscription** object to the **Subscriber**. It is through the **Subscription** that the **Subscriber** can manage its subscription.
```java
public interface Subscription {
		void request(long n);
		void cancel();
}
```

As for the **Processor** interface, it's a combination of **Subscriber** and **Publisher**. As a **Subscriber**, a **Processor** will receive data and process it in some way. Then it will switch hats and act as a **Publisher** to publish the results to its **Subscriber**s.

# Project Reactor
Project Reactor is an implementation of the Reactive Streams specification that provides a functional API for composing Reactive Streams. Also, Reactor is the foundation for Spring 5's reactive programming model.

```java
Mono.just("Craig")
		.map(n -> n.toUpperCase())
		.map(cn -> "Hello, " + cn + "!")
		.subscribe(System.out::println);
```


