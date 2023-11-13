# What Are Threads?
You should decouple the *task* that is to be run in parallel from the *mechanism* of running it. If you have many tasks, it is too expensive to create a separate thread for each of them. Instead, you can use a thread pool.

Do not call the **run** method of the **Thread** class or the **Runnable** object. Calling the **run** method directly merely executes the task in the same thread, no new thread is started. Instead, call the **Thread.start** method. It creates a new thread that executes the **run** method.

# Thread States
Threads can be in one of six states:
- New
- Runnable
- Blocked
- Waiting
- Timed waiting
- Terminated

When a thread is in the *new* state, the program has not started executing code inside of it. A certain amount of bookkeeping needs to be done before a thread can run.

Once you invoke the **start** method, the thread is in the *runnable* state. A runnable thread may or may not actually running. It is up to the operating system to give the thread time to run.

Once a thread is running, it doesn't necessarily keep running. In fact, it is desirable that running threads occasionally pause so that other threads have a chance to run. The details of thread scheduling depend on the services that the operating system provides. Preemptive scheduling systems give each runnable thread a slice of time to perform its task. When that slice of time is exhausted, the operating system preempts the thread and gives another thread an opportunity to work. All modern desktop and server operating systems use preemptive shceduling. However, small devices such as cell phones may use cooperative scheduling. In such a device, a thread loses control only when it calls the **yield** method, or when it is blocked or waiting.

When the thread tries to acquire an intrinsic object lock (but not a **Lock** in the **java.util.concurrent** library) that is currently held by another thread, it becomes *blocked*. The thread becomes unblocked when all other threads have relinquished the lock and the thread scheduler has allowed this thread to hold it.

When the thread waits for another thread to notify the scheduler of a condition, it enters the *waiting* state. This happens by calling the **Object.wait** or **Thread.join** method, or by waiting for a **Lock** or **Condition** in the **java.util.concurrent** library. In practice, the difference between the blocked and waiting state is not significant.

A thread is terminated for one of two reasons:
- it dies a natural death because the **run** method exists normally
- it dies abruptly because an uncaught exception terminates the **run** method
  
# Interrupting Threads
The **interrupt** method can be used to request termination of a thread. When the **interrupt** method is called on a thread, the *interrupted status* of the thread is set. This is a boolean flag that is present in every thread. Each thread should occasionally check whether it has been interrupted.

When the **interrupt** method is called on a thread that blocks on a call such as **sleep** or **wait**, the blocking call is terminated by an **InterruptedException**.

There is no language requirement that a thread which is interrupted should terminate. Interrupting a thread simply grabs its attention. The interrupted thread can decide how to react to the interruption.

# Daemon Threads
A daemon thread is simply a thread that has no other role in life than to serve others. When only daemon threads remain, the virtual machine exists. There is no point in keeping the program running if all remaining threads are daemons.

# Uncaught Exception
The **run** method of a thread cannot throw any checked exceptions, but it can be terminated by an unchecked exception. In that case, the thread dies. Just before the thread dies, the exception is passed to a handler for uncaught exception.

# ReentrantLock
It is called *reentrant* because a thread can repeatedly acquire a lock that it already owns. The lock has a *hold count* that keeps track of the nested calls to the **lock** method. The thread has to call **unlock** for every call to **lock** in order to relinquish the lock. Because of this feature, code protected by a lock can call another method that uses the same locks.