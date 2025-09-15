In the ancient past, computers didn't have operating systems; they executed a single program from beginning to end, and that program had direct access to all the resources of the machine.

Operating systems evolved to allow more than one program to run at once, running individual programs in *processes*: isolated, indenpendently executing programs to which the operating system allocates resources.

In early timesharing systems, each process was a virtual von Neumann computer; it had a memory space storing both instructions and data, executing instructions sequentially according to the semantics of the machine language, and interacting with the outside world via the operating system through a set of I/O primitives.

The same concerns (resource utilization, fairness, and convenience) that motivated the development of processes also motivated the development of *threads*. Threads allow multiple streams of program control flow to coexist within a process. They share process-wide resources such as memory and file handles, but each thread has its own program counter, stack, and local variables. Threads are sometimes called *lightwight processes*, and most modern operating systems treat threads, not processes, as the basic units of scheduling. 

Since threads share the memory address space of their owing process, all threads within a process have access to the same variables and allocate objects from the same heap, which allows finer-grained data sharing than inter-process meachanisms. But without explicit synchronization to coordinate access to shared data, a thread may modify variables that another thread is in the middle of using, with unpredictable results.

# Benefits of threads
## Exploiting multiple processors
When properly designed, multithreaded programs can improve throughout by utilizing available processor resources more effectively.

## Simplicity of modeling
A program that processes one type of task sequentially is simpler to write, less error-prone, and easier to test than one managing multiple different types of tasks at once. Assigning a thread to each type of task or to each element in a simulation affords the illusion of sequentiality and insulates domain logic from the details of scheduling, interleaved operations, asynchronous I/O, and resource waits. A complicated, asynchronous workflow can be decomposed into a number of simpler, synchronous workflows each running in a separate thread, interacting only with each other at specific synchronization points.

## Simplified handling of asynchronous events
A server application that accepts socket connections from multiple remote clients may be easier to develop when each connection is allocated its own thread and allowed to use synchronous I/O.

Historically, operating systems placed relatively low limits on the number of threads that a process could create, as few as several hundred (or even less). As a result, operating systems developed efficient facilities for multiplexed I/O, such as the Unix **select** and **poll** system calls, and to access these facilities, the Java class libraries acquired a set of packages (java.nio) for nonblocking I/O. However, operating system support for large numbers of threads has improved significantly, making the thread-per-client model practical even for large numbers of clients on some platforms.

## More responsive user interfaces
GUI applications used to be single-threaded, which means that you had to either frequently poll throughout the code for input events (which is messy and intrusive) or execute all application code indirectly through a "main event loop".

Modern GUI frameworks, such as the AWT and Swing toolkits, replace the main event loop with an *event dispatch thread* (EDT).

# Risks of threads
## Safety hazards
Thread safety can be unexpectedly subtle because, in the absence of sufficient synchronization, the ordering of operations in multiple threads is unpredictable and sometimes surprising.

Because threads share the same memory address space and run concurrently, they can access or modify variables that other threads might be using. This is a tremendous convenience, because it makes data sharing much easier than would other inter-thread communications mechanisms. But it is also a significant risk: threads can be confused by having data change unexpectedly. For a multithreaded program's behavior to be predictable, access to shared variables must be properly coordinated so that threads do not interfere with one another.

In the absence of synchronization, the compiler, hardware, and runtime are allowed to take substantial liberties with the timing and ordering of actions, such as caching variables in registers or processor-local caches where they are temporarily (or even permanently) invisible to other threads.

```java
@NotThreadSafe
public class UnsafeSequence {
  private int value;

  public int getNext() {
    return value++;
  }
}
```

```java
@ThreadSafe
public class SafeSequence {
  @GuardedBy("this")
  private int value;

  public synchronized int getNext() {
    return value++;
  }
}
```

## Liveness hazards
A liveness failure occurs when an activity gets into a state such that it is permanently unable to make forward progress. One form of liveness failure that can occur in sequential programs is an inadvertent infinite loop. The use of threads introduces additional liveness risks.

## Performance hazards
Performance issues subsume a broad range of problems, including poor service time, responsiveness, throughput, resource consumption, or scalability.

In well designed concurrent applications the use of threads is a net performance gain, but threads nevertheless carry some degree of runtime overhead. Context switches, when the scheduler suspends the active thread temporarily so another thread can run, are more frequent in applications with many threads, and have significant costs: saving and restoring execution context, loss of locality, and CPU time spent scheduling threads instead of running them. When threads share data, they must use synchronization mechanisms that can inhibit compiler optimizations, flush or invalidate memory caches, and create synchronization traffic on the shared memory bus. All these factors introduce additional performance costs.

# Threads are everywhere
Every Java application uses threads. When the JVM starts, it creates threads for JVM housekeeping tasks (garbage collection, finalization) and a main thread for running the **main** method. The AWT and Swing user interface frameworks create threads for managing user interface events. **Timer** creates threads for executing deferred tasks. Component frameworks, such as servlets and RMI creatte pools of threads and invoke component methods in these threads.

Frameworks introduce concurrency into applications by calling application components from framework threads. Components invariably access application state, thus requiring that all code paths accessing that state be thread-safe.