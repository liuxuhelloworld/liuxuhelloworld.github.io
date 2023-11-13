Concurrent programming includes all the tools and techniques to have tasks or processes running at the same time in a computer, communicating and synchronizing between them without data loss or inconsistency.

Synchronization can be classified as two kinds:
- control synchronization: when, for example, one task depends on the end of another task, the second task can't start before the first has finished
- data access synchronization: when two or more tasks have access to a shared variable and only one of the tasks can access the variable at any given time

The granularity of your concurrent algorithm is important. If you have a coarse-grained granularity (big tasks with low intercommunication), the overhead due to synchronization will be low. However, maybe you won't benefit all the cores of your system. If you have a fine-grained granularity (small tasks with high intercommunication), the overhead due to synchronization will be high and maybe the throughput of your algorithm won't be good.

A piece of code (or a method or an object) is **thread-safe** if all the users of shared data are protected by synchronization mechanisms, a nonblocking compare-and-swap (CAS) primitive or data is immutable, so you can use that code in a concurrent application without any problem.