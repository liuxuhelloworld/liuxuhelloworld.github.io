We have two conflicting goals here. The first general rule is to create objects sparingly and to discard them as quickly as possible. Using less memory is the best way to improve the efficiency of the garbage collector. On the other hand, frequently recreating some kinds of objects can lead to worse overall performance (even if GC performance improves). If those objects are instead reused, programs can see substantial performance gains. Objects can be reused in a variety of ways, including thread-local variables, special object references, and object pools. Reusing objects means they will be long-lived and impact the garbage collector, but when they are reused judiciously, overall performance will improve.

# Heap Analysis
```bash
jcmd pid GC.class_histogram

jmap -histo:live pid
```

Histograms are a quick way to look at the number of objects within an application without doing a full heap dump (since heap dumps can take a while to analyze, and they consume a large amount of disk space).

```bash
jcmd pid GC.heap_dump /xxx/xxx/heap_dump.hprof

jmap -dump:live,file=/xxx/xxx/heap_dump.hprof pid
```

The first-pass analysis of a heap generally involves retained memory. The retained memory of an object is the amount of memory that would be freed if the object itself were eligible to be collected.

Two other useful terms for memory analysis are *shallow* and *deep*. The *shallow size* of an object is the size of the object itself. If the object contains a reference to another object, the 4 or 8 bytes of the reference is included, but the size of the target object is not included. The *deep size* of an object includes the size of the object it referenes. The difference between the deep size of an object and the retained memory of an object lies in objects that are otherwise shared.

```bash
-XX:+HeapDumpOnOutOfMemoryError

-XX:HeapDumpPath=path

-XX:+HeapDumpAfterFullGC

-XX:+HeapDumpBeforeFullGC
```

Out-of-memory errors can occur unpredictably, making it difficult to know when to get a heap dump. The above flags can help.

# Out-of-Memory Errors
The JVM throws an *out-of-memory* error under these circumstances:
- no native memory is available for the JVM
- the metaspace is out of memory
- the Java heap itself is out of memory: the application cannot create any additional objects for the given heap size
- the JVM is spending too much time performing GC

If it is the out-of-heap-memory error, the application may simply need more heap space: the number of live objects that it is holding onto cannot fit in the heap space configured for it. Or, the application may have a memory leak: it continues to allocate additional objects without allowing other objects to go out of scope.

Collection classes are the most frequent cause of a memory leak: the application inserts items into the collection and never frees them. The best way to overcome this is to change the application logic such that items are proactively discarded from the collection when they are no longer needed. Alternatively, a collection that uses weak or soft references can automatically discard the items when nothing else in the application is referencing them, but those collections come with a cost.

For both the metaspace and the regular heap, out-of-memory errors most frequently occur because of memory leaks; heap analysis tools can help to find the root cause of the leak.

# Using Less Memory
Using less memory means the heap will fill up less often, requiring fewer GC cycles. The effect can multiply: fewer collections of the young generation means the tenuring age of an object is increased less often, meaning that the object is less likely to be promoted into the old generation. Hence, the number of full GC cycles (or concurrent GC cycles) will be reduced. And if those full GC cycles can clear up more memory, they will also occur less frequently.

## reducing object size
Objects occupy a certain amount of heap memory, so the simplest way to use less memory is to make objects smaller. The size of an object can be decreased by reducing the number of instance variables it holds and by reducing the size of those variables.

The size of an object also includes some invisible object header fields. For a regular object, the size of the header fields is 8 bytes on a 32-bit JVM, and 16 bytes on a 64-bit JVM. For an array, the size of the header fields is 16 bytes on a 32-bit JVM or a 64-bit JVM with a heap of less than 32 GB, and 24 bytes otherwise.

Eliminating instance fields in an object can help make the object smaller, but a gray area exists: what about object fields that hold the result of a calculation based on pieces of data? This is the classic computer science trade-off time versus space: is it better to spend the memory to store the value or better to spend the time to calculate the value as needed? This is definitely a your-mileage-may-vary situation and the point along the time/space continuum where it makes sense to switch between using the memory to cache a value and recalculating the value will depend on many factors.