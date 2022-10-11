# Garbage Collection Overview

One of the most attractive features of programming in Java is that developers needn't explicitly manage the life cycle of objects: objects are created when needed, and when the object is no longer in use, the JVM automatically frees the object.

At a basic level, GC consists of finding objects that are in use and freeing the memory associated with the remaining objects (those that are not in use). 

The JVM must periodically search the heap for unused objects. It does this by starting with objects that are GC roots, which are objects that are accessible from outside the heap. That primarily includes thread stacks and system classes. Those objects are always reachable, so then the GC algorithm scans all objects that are reachable via one of the root objects. Objects that are reachable via a GC root are live objects; the remaining unreachable objects are garbage (even if they maintain references to live objects or to each other).

When the GC algorithm finds unused objects, the JVM can free the memory occupied by those objects and use it to allocate additional objects. However, it is usually insufficient simply to keep track of that free memory and use it for future allocations; at some point, memory must be compacted to prevent memory fragmentation.

It is simpler to perform these operations if no application threads are running while the garbage collector is running. When GC threads track object references or move objects around in memory, they must make sure that application threads are not using those objects. This is particularly true when GC moves objects around: the memory location of the object changes during that operation, and hence no application threads can be accessing the object. The pauses when all application threads are stopped are called *stop-the-world pauses*. These pauses generally have the greatest impact on the performance of an application, and minimizing those pauses is one important consideration when tuning GC.

# Generational Garbage Collectors

Most garbage collectors work by splitting the heap into generations. These are called the *old generation* and the *yound generation*. The young generation is further divided into sections known as *eden* and *survior spaces*.

The rationale for having separate generations is that many objects are used for a very short period of time. The garbage collector is designed to take advantage of the fact that many objects are only used temporarily. This is where the generational design comes in.

Objects are first allocated in the young generation, which is a subset of the entire heap. When the young generation fills up, the garbage collector will stop all the application threads and empty out the young generation. Objects that are no longer in use are discarded, and objects that are still in use are moved elsewhere. This operation is called a *minor GC* or *young GC*.

Because the young generation is only a portion of the entire heap, processing it is faster than processing the entire heap. The application threads are stopped for a much shorter period of time than if the entire heap were processed at once. It is almost always a big advantage to have the shorter pauses even though they will be more frequent.

Objects are allocated in eden (which encompasses the vast majority of the young generation). When the young generation is cleared during a collection, all objects in eden are either moved or discarded: objects that are not in use can be discarded, and objects in use are moved either to one of the survior spaces or to the old generation. Since all surviving objects are moved, the young generation is automatically compacted when it is collected: at the end of the collection, eden and one of the survior spaces are empty, and the objects that remain in the young generation are compacted within the other survior space.

As objects are moved to the old generation, eventually it too will fill up, and the JVM will need to find any objects within the old generation that are no longer in use and discard them. This is where GC algorithms have their biggest differences. The simpler algorithms stop all application threads, find the unused objects, free their memory, and then compact the heap. This process is called a *full GC*, and it generally causes a relatively long pause for the application threads. One the othre hand, it is possible, though more computationally complex, to find unused objects while application threads are running. Because the phase where they scan for unused objects can occur without stopping application threads, these algorithms are called *concurrent collectors*. When using a concurrent collector, an application will typically experience fewer (and much shorter) pauses. The biggest trade-off is that the application will use more CPU overall.

# Serial Garbage Collector

```
-XX:+UseSerialGC
```

The *serial garbage collector* is the simplest of the collectors. 

The serial collector uses a single thread to process the heap. It will stop all application threads as the heap is processed (for either a minor or full GC). During full GC, it will fully compact the old generation.

On a machine with a single CPU, the JVM defaults to use the serial collector. This includes virtual machines with one CPU, and Docker containers that are limited to one CPU. In these environments, the serial collector is usually a good choice, but at times G1 GC will give better results.


# Throughput Garbage Collector

```
-XX:+UseParallelGC
-XX:+UseParallelOldGC (obsolete)
```

In JDK 8, the *throughput collector* is the default collector for any 64-bit machine with two or more CPUs.

The throughput collector uses multiple threads to collect the young generation, which makes minor GCs much faster than when the serial collector is used. It uses multiple threads to process the old generation as well. The throughput collector stops all application threads during both minor and full GCs, and it fully compacts the old generation during a full GC.

# G1 (Garbage First) Garbage Collector

```
-XX:+UseG1GC
```

The *G1 garbage collector* uses a concurrent collection strategy to collect the heap with minimal pauses. It is the default collector in JDK 11 and later for 64-bit JVMs on machines with two or more CPUs. It is functional in JDK 8 as well, particularly in later builds of JDK 8.

G1 GC divides the heap into regions, but it still considers the heap to have two generations. Some of those regions make up the young generation, and the young generation is still collected by stopping all application threads and moving all objects that are alive into the old generation or the survior spaces. (This occurs using multiple threads).

In G1 GC, the old generation is processed by background threads that don't need to stop the application threads to perform most of their work. Because the old generation is divided into regions, G1 GC can clean up objects from the old generation by copying from one region into another, which means that it (at least partially) compacts the heap during normal processing. This helps keep G1 GC heaps from becoming fragmented, although that is still possible.

The trade-off for avoiding the full GC cycles is CPU time: the (multiple) background threads G1 GC uses to process the old generation must have CPU cycles available at the same time the application threads are running.

# CMS (Concurrent Mark Sweep) Garbage Collector

```
-XX:+UseConcMarkSweepGC
-XX:+UseParNewGC (obsolete)
```

The *CMS garbage collector* was the first concurrent collector. Like other algorithms, CMS stops all application threads during a minor GC, which it performs with multiple threads.

CMS is officially deprecated in JDK 11 and beyond, and its use in JDK 8 is discouraged. From a practical standpoint, the major flaw in CMS is that it has no way to compact the heap during its background processing. If the heap becomes fragmented, CMS must stop all application threads and compact the heap, which defeats the purpose of a concurrent collector.