Delagation is one of the most effective strategies for creating thread-safe classes: just let existing thread-safe classes manage all the state. The Java platform libraries include a rich set of concurrent building blocks, such as thread-safe collections and a variety of *synchronizers* that can coordinate the control flow of cooperating threads.

# Synchronized collections
The *synchronized collection classes* include **Vector** and **Hashtable**, part of the original JDK, as well as their cousins added in JDK 1.2, the synchronized wrapper classes created by the **Collections.synchronizedXXX** factory methods.

These classes achieve thread safety by encapsulating their state and synchronizing every public method so that only one thread at a time can access the collection state.

The synchronized collections are thread-safe, but you may sometimes need to use additional client-side locking to guard compound actions. With a synchronized collection, compound actions are still technically thread-safe even without client-side locking, but they may not behave as you might expect when other threads can concurrently modify the collection.

```java
	public static Object getLast(Vector list) {
		synchronized (list) {
			int lastIndex = list.size() - 1;
			return list.get(lastIndex);
		}
	}

	public static void deleteLast(Vector list) {
		synchronized (list) {
			int lastIndex = list.size() - 1;
			list.remove(lastIndex);
		}
	}

	public static void iterate(Vector list) {
		synchronized (list) {
			for (int i = 0; i < list.size(); i++) {
				doSomething(list.get(i));
			}
		}
	}
```

The iterators returned by the synchronized collections are not designed to deal with concurrent modification, and they are *fail-fast*, meaning that if they detect that the collection has changed since iteration began, they throw the unchecked **ConcurrentModificationException**. They are implemented by associating a modification count with the collection: if the modification count changes during iteration, **hasNext** or **next** throws **ConcurrentModificationException**. However, this check is done without synchronization, so there is a risk of seeing a stale value of the modification count and therefore that the iterator does not realize a modification has been made.

An alternative to locking the collection during iteration is to clone the collection and iterate the copy instead. Cloning the collection has an obvious performance cost; whether this is favorable tradeoff depends on many factors including the size of the collection, how much work is done for each element, the relative frequency of iteration compared to other collection operations, and responsiveness and throughput requirements.

# Concurrent collections
Java 5.0 improves on the synchronized collections by providing several concurrent collection classes. The concurrent collections are designed for concurrent access from multiple threads. Replacing synchronized collections with concurrent collections can offer dramatic scalability improvements with little risk.

## ConcurrentHashMap
A replacement for synchronized hash-based **Map** implementations.

**ConcurrentHashMap** is a hash-based **Map** like **HashMap**, but it uses an entirely different locking strategy that offers better concurrency and scalability. Instead of synchronizing every method on a common lock, restricting access to a single thread at a time, it uses a finer-grained locking mechanism called *locking striping* to allow a greater degree of shared access. Arbitrarily many reading threads can access the map concurrently, readers can access the map concurrently with writers, and a limited number of writers can modify the map concurrently.

**ConcurrentHashMap** further improve on the synchronized collection classes by providing iterators that do not throw **ConcurrentModificationException**, thus eliminating the need to lock the collection during iteration. The iterators returned by **ConcurrentHashMap** are *weakly consistent* instead of fail-fast. A weakly consistent iterator can tolerate concurrent modification, traverses elements as they existed when the iterator was constructed, and may (but is not guaranteed to) reflect modifications to the collection after the construction of the iterator.

The one feature offered by the synchronized **Map** implementations but not by **ConcurrentHashMap** is the ability to lock the map for exclusive access. Since a **ConcurrentHashMap** cannot be locked for exclusive access, we cannot use client-side locking to create new atomic operations such as put-if-absent. Instead, a number of common compound operations such as put-if-absent, remove-if-equal, and replace-if-equal are provided as atomic operations.

Because it has so many advantages and so few disadvantages compared to **Hashtable** or **synchronizedMap**, replacing synchronized **Map** implementations with **ConcurrentHashMap** in most cases results only in better scalability.

## CopyOnWriteArrayList
**CopyOnWriteArrayList** is a concurrent replacement for a synchronized **List** that offers better concurrency in some common situations and eliminates the need to lock or copy the collection during iteration.

The copy-on-write collections derive their thread safety from the fact that as long as an effectively immutable object is properly published, no further synchronization is required when accessing it. They implement mutablity by creating and republishing a new copy of the collection every time it is modified. The iterators returned by the copy-on-write collections do not throw **ConcurrentModificationException** and return the elements exactly as they were at the time the iterator was created, regardless of subsequent modifications.

Obviously, there is some cost to copying the backing array every time the collection is modified, especially if the collection is large; the copy-on-write collections are reasonable to use only when iteration is far more common than modification.

# Blocking queues
Blocking queues support the *producer-consumer* design pattern. In a producer-consumer design built around a blocking queue, producers place data onto the queue as it becomes available, and consumers retrieve data from the queue when they are ready to take the appropriate action. One of the most common producer-consumer designs is a thread pool coupled with a work queue; this pattern is embodied in the **Executor** task execution framework.

Blocking queues are a powerful resource management tool for building reliable applications: they make your program more robust to overload by throttling activities that threaten to produce more work than can be handled.

The producer-consumer pattern also enables several performance benefits. Producers and consumers can execute concurrently; if one is IO-bound and the other is CPU-bound, executing them concurrently yields better overall throughput than executing them sequentially.

## FileCrawler and Indexer
```java
public class FileCrawler implements Runnable {
	private final BlockingQueue<File> fileQueue;
	private final File root;

	public FileCrawler(BlockingQueue<File> fileQueue, File root) {
		this.fileQueue = fileQueue;
		this.root = root;
	}

	@Override
	public void run() {
		try {
			crawl(root);
		} catch (InterruptedException e) {
			Thread.currentThread().interrupt();
		}
	}

	private void crawl(File root) throws InterruptedException {
		File[] files = root.listFiles();
		if (files != null) {
			for (File file : files) {
				if (file.isDirectory()) {
					crawl(file);
				} else {
					fileQueue.put(file);
				}
			}
		}
	}
}

public class Indexer implements Runnable {
	private final BlockingQueue<File> fileQueue;

	public Indexer(BlockingQueue<File> fileQueue) {
		this.fileQueue = fileQueue;
	}

	@Override
	public void run() {
		try {
			while (true) {
				File file = fileQueue.take();
				System.out.println(Thread.currentThread().getName() + ":" + file.getCanonicalPath());
			}
		} catch (InterruptedException | IOException e) {
			Thread.currentThread().interrupt();
		}
	}
}

public class FileCrawlerAndIndexerTest {
	public static void main(String[] args) {
		BlockingQueue<File> fileQueue = new LinkedBlockingQueue<>(100);
		File root = new File("/Users/tiger/liuxu");

		new Thread(new FileCrawler(fileQueue, root)).start();

		for (int i = 0; i < 10; i++) {
			new Thread(new Indexer(fileQueue)).start();
		}
	}
}
```

## serial thread confinement
For mutable object, producer-consumer designs and blocking queues facilitate *serial thread confinement* for handling off ownership of objects from producers to consumers. A thread-confined object is owned exclusively by a single thread, but that ownership can be transferred by publishing it safely where only one other thread will gain access to it and ensuring that the publishing thread does not access it after the handoff.

Object pools exploit serial thread confinement, lending an object to a requesting thread. As long as the pool contains sufficient internal synchronization to publish the pooled object safely, and as long as the clients do not themselves publish the pooled object or use it after returning it to the pool, ownership can be transferred safely thread to thread.

## deques and work stealing
A **Deque** is a double-ended queue that allows efficient insertion and removal from both the head and the tail.

Just as blocking queues lend themselves to the producer-consumer pattern, deques lend themselves to a related pattern called *work stealing*. A producer-consumer design has one shared work queue for all consumers; in a work stealing design, every consumer has its own deque. If a consumer exhausts the work in its own deque, it can steal work from the tail of someone else's deque.

Work stealing can be more scalable than a traditional producer-consumer design because workers don't contend for a shared work queue; most of the time they access only their own deque, reducing contention. When a worker has to access another's queue, it does so from their tail rather than the head, further reducing contention.

Work stealing is well suited to problems in which consumers are also producers, when performing a unit of work is likely to result in the identification of more wok. For example, processing a page in a web crawler usually results in the identification of new pages to be crawled. Similarly, mamy graph-exploring algorithms, such as marking the hap during garbage collection, can be efficiently parallelized using work stealing. When a work identifies a new unit of work, it places it at the end of its own deque; when its deque is empty, it looks for work at the end of someone else's deque, ensuring that each worker stays busy.

# Blocking and interruptible methods
Threads may block for several reasons: waiting for I/O completion, waiting to acquire a lock, waiting to wake up from **Thread.sleep**, or waiting for the result of a computation in another thread. When a thread blocks, it is usually suspended and placed in one of the blocked thread states (BLOCKED, WAITING, or TIMED_WAITING).

When a method can throw **InterruptedException**, it is telling you that it is a blocking method, and further if it is interrupted, it will make an effort to stop blocking early.

Interruption is a cooperative mechanism. One thread cannot force another to stop what it is doing and do something else; when thread A interrupts thread B, A is merely requesting that B stop what it is doing when it gets to a convenient stoping point, if it feels like it. One of the most sensible use for interruption is to cacel an activity. Blocking methods that are responsive to interruption make it easier to cancel long-running activities on a timely basis.

When your code calls a method that throws **InterruptedException**, then your method is a blocking method too, and must have a plan for responding to interruption. For library code, there are basically two choices:
- propagate the **InterruptedException**: this is often the most sensible policy if you can get away with it, just propagate the **InterruptedException** to your caller. This could involve not catching **InterruptedException**, or catching it and throwing it again after performing some brief activity-specific cleanup
- restore the interrupt: sometimes you cannot throw **InterruptedException**, for instance when your code is part of a **Runnable**. In these situations, you must catch **InterruptedException** and restore the interrupted status by calling **interrupt** on the current thread, so the caller higher up the call stack can see that an interrupt was issued.

# Synchronizers
A *synchronizer* is any object that coordinates the control flow of threads based on its state.

Blocking queues can act as synchronizers; other types of synchronizers include semaphores, barriers, and latches.

## latch
A *latch* is a synchronizer that can delay the progress of threads until it reaches its terminal state. A latch acts as a gate: until the latch reaches the terminal state the gate is closed and no thread can pass, and in the terminal state the gate opens, allowing all threads to pass.

Latches are single-use objects; once a latch enters the terminal state, it cannot be reset.

**CountDownLatch** is a flexible latch implementation; it allows one or more threads to wait for a set of events to occur. The latch state consists of a counter initialized to a positive number, representing the number of events to wait for. The **countDown** method decrements the counter, indicating that an event has occured, and the **await** method wait for the counter to reach zero, which happens when all the events have occurred.

```java
public class CountDownLatchExample {
	public long timeTasks(int nThreads, final Runnable task) throws InterruptedException {
		final CountDownLatch startGate = new CountDownLatch(1);
		final CountDownLatch endGate = new CountDownLatch(nThreads);

		for (int i = 0; i < nThreads; i++) {
			Thread t = new Thread() {
				public void run() {
					try {
						startGate.await();
						try {
							task.run();
						} finally {
							endGate.countDown();
						}
					} catch (InterruptedException e) {

					}
				}
			};
			t.start();
		}

		long start = System.nanoTime();
		startGate.countDown();
		endGate.await();
		long end = System.nanoTime();

		return end - start;
	}
}
```

## FutureTask
**FutureTask** implements **Future**, which describes an abstract result-bearing computation. 

A computation represented by a **FutureTask** is implemented with a **Callable** and can be in one of three states: waiting to run, running, or completed. Completion subsumes all the ways a computation can complete, including normal completion, cancellation, and exception. Once a **FutureTask** enters the completed state, it stays in that state forever.

The behavior of **Future.get** depends on the state of the task. If it is completed, **get** returns the result immediately, and otherwise blocks until the task transitions to the completed state and then returns the result or throws an exception. **FutureTask** conceys the result from the thread executing the computation to the thread retrieving the result; the specification of **FutureTask** guarantees that this transfer constitutes a safe publication of the result.

**FutureTask** is used by the **Executor** framework to represent asynchronous tasks, and can also be used to represent any potentially lenghy computation that can be started before the results are needed.

```java
public class FutureTaskExample {
	private final FutureTask<Resource> futureTask = new FutureTask<>(new Callable<Resource>() {
		@Override
		public Resource call() throws Exception {
			return new Resource();
		}
	});

	private final Thread thread = new Thread(futureTask);

	public void start() {
		thread.start();
	}

	public Resource get() {
		try {
			futureTask.get();
		} catch (ExecutionException e) {
			e.printStackTrace();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		return null;
	}
}
```

## semaphores
Counting semaphores are used to control the number of activities that can access a certain resource or perform a given action at the same time. Counting semaphores can be used to implement resource pools or to impose a bound on a collection.

A **Semaphore** manages a set of virtual *permits*; the initial number of permits is passed to the **Semaphore** constructor. Activities can acquire permits as long as some remain and release permits when they are done with them. If no permit is avaiable, **acquire** blocks until one is (or until interrupted or the operation times out). The **release** method returns a permit to the semaphore.

A binary semaphore can be used as a *mutex* with nonreentrant locking semantics; whoever holds the sole permit holds the mutex.

Semaphores are useful for implementing resource pools such as database connection pools. If you initialize a **Semaphore** to the pool size, **acquire** a permit before trying to fetch a resource from the pool, and **release** the permit after putting a resource back in the pool, **acquire** blocks until the pool becomes nonempty.

Similarly, you can use a **Semaphore** to turn any collection into a blocking bounded collection.

```java
public class BlockingBoundedHashSet<E> {
	private final Set<E> set;
	private final Semaphore semaphore;

	public BlockingBoundedHashSet(int bound) {
		set = Collections.synchronizedSet(new HashSet<>());
		semaphore = new Semaphore(bound);
	}

	public boolean add(E o) throws InterruptedException {
		semaphore.acquire();
		boolean added = false;
		try {
			added = set.add(o);
			return added;
		} finally {
			if (!added) {
				semaphore.release();
			}
		}
	}

	public boolean remove(Object o) {
		boolean removed = set.remove(o);
		if (removed) {
			semaphore.release();
		}

		return removed;
	}
}
```

The semaphore is initialized to the desired maximum size of the collection. The **add** operation acquires a permit before adding the item into the underlying collection. If the underlying **add** operation does not actually add anything, it releases the permit immediately. Similarly, a succussful **remove** operation releases a permit, enabling more elements to be added.

## barriers
Barriers are similar to latches in that they block a group of threads until some event has occurred. The key difference is that with a barrier, all the threads must come together at a barrier point at the same time in order to proceed. Latches are for waiting for events, barriers are for waiting for other threads.

**CyclicBarrier** allows a fixed number of parties to rendezvous repeatedly at a *barrier point* and is useful in parallel iterative algorithms that break down a problem into a fixed number of independent subproblems. Threads call **await** when they reach the barrier point, and **await** blocks until all the threads have reached the barrier point. If all threads meet at the barrier point, the barrier has been successfully passed, in which case all threads are released and the barrier is reset so it can be used again. If the barrier is successfully passed, **await** returns a unique arrival index for each thread, which can be used to elect a leader that takes some special action in the next iteration. **CyclicBarrier** also lets you pass a *barrier action* to the constructor; this is a **Runnable** that is executed when the barrier is successfully passed but before the blocked threads are released.

Barriers are often used in simulations, where the work to calculate one step can be done in parallel but all the work associated with a given step must complete before advancing to the next step.

```java
public class CyclicBarrierExample {
	private final List<Integer> box;
	private final CyclicBarrier barrier;
	private final Worker[] workers;

	public CyclicBarrierExample(List<Integer> box) {
		this.box = box;

		int count = Runtime.getRuntime().availableProcessors();

		this.barrier = new CyclicBarrier(count, new Runnable() {
			@Override
			public void run() {
				System.out.println(box);
				box.clear();
			}
		});

		this.workers = new Worker[count];
		for (int i = 0; i < count; i++) {
			workers[i] = new Worker(box);
		}
	}

	public void start() {
		for (int i = 0; i < workers.length; i++) {
			new Thread(workers[i]).start();
		}
	}

	public static void main(String[] args) {
		List<Integer> box = new ArrayList<>();
		CyclicBarrierExample cyclicBarrierExample = new CyclicBarrierExample(box);
		cyclicBarrierExample.start();
	}

	private class Worker implements Runnable {
		private final List<Integer> box;

		public Worker(List<Integer> box) {
			this.box = box;
		}

		@Override
		public void run() {
			int val = 1;
			while (val < 10) {
				box.add(val * val);
				try {
					barrier.await();
				} catch (InterruptedException e) {
					return;
				} catch (BrokenBarrierException e) {
					return;
				}
				val++;
			}
		}
	}
}
```

Another form of barrier is **Exchanger**, a two-party barrier in which the parties exchange data at the barrier point. Exchangers are useful when the parties perform asymmetric activities, for example when one thread fills a buffer with data and the other thread consumes the data from the buffer; these threads could use an **Exchanger** to meet and exchange a full buffer for an empty one.