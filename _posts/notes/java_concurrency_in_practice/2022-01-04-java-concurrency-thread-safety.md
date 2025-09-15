Writing thread-safe code is about managing access to state, and in particular to shared, mutable state. By shared, we mean that a variable could be accessed by multiple threads; by mutable, we mean that its value could change during its life time.

Whether an object needs to be thread-safe depends on whether it will be accessed from multiple threads. Making an object thread-safe requires using synchronization to coordinate access to its mutable state.

**Whenever more than one thread accesses a given state variable, and one of them might write to it, they all must coordinate their access to it using synchronization.**

For the safety of mutable state variable, you can:
- don't share the state variable across threads
- make the state variable immutable
- use synchronization whenever accessing the state variable

Thread safety is about state, and it can only be applied to the entire body of code that encapsulates its state, which may be an object or an entire program. A program that consists entirely of thread-safe classes may not be thread-safe, and a thread-safe program may contain classes that are not thread-safe.

A class is thread-safe if it behaves correctly when accessed from multiple threads, regardless of the scheduling or interleaving of the execution of those threads by the runtime environment, and with no additional synchronization or other coordination on the part of the calling code. No set of operations performed sequentially or concurrently on instances of a thread-safe class can cause an instance to be in an invalid state. Thread-safe classes encapsulate any needed synchronization so that clients need not provide their own.

When designing thread-safe classes, good object-oriented techniques, such as encapsulation, immutability, and clear specification of invariants, are your best friends.

# Stateless object
A Stateless object is always thread-safe, because the actions of a thread accessing a stateless object cannot affect the correctness of operations in other threads.

```
@ThreadSafe
public class StatelessObject {
	public void service() {
		System.out.println("hahaha");
	}
}
```

# Race condition
The possibility of incorrect results in the presence of unlucky timing is so important in concurrent programming that it has a name: a race condition.

A race condition occurs when the correctness of a computation depends on the relative timing or interleaving of multiple threads by the runtime, in other words, when getting the right answer relies on lucky timing.

## check-then-act
The most common type of race condition is check-then-act, where a potentially stale observation is used to make a decision on what to do next.

For example, you observe something to be true (file X doesn't exist) and then take action based on that observation (create X), but in fact the observation could have become invalid between the time you observed it and the time you acted on it (someone else created X in the meantime), causing a problem (unexpected exception, overwritten data, file corruption).

A common idiom that uses check-then-act is *lazy initialization*.

```
@NotThreadSafe
public class LazyInitRace {
	private static Resource instance = null;

	public static Resource getInstance() {
		if (instance == null) {
			instance = new Resource();
		}

		return instance;
	}

	public static void main(String[] args) throws InterruptedException {
		Runnable runnable = () -> {
			Resource resource = getInstance();
			System.out.println(resource);
		};

		new Thread(runnable).start();
		new Thread(runnable).start();

		sleep(1000);
	}
}
```

## read-mofidy-write
Another sort of race condition is read-modify-write. 

Read-modify-write operations, like incrementing a counter, define a transformation of an object's state in terms of its previous state. To increment a counter, you have to know its previous value and make sure no one else changes or uses that value while you are in mid-update.

```
@NotThreadSafe
public class UnsafeCounter {
	private volatile long count = 0;

	public long getCount() {
		return count;
	}

	public void service() {
		++count;
	}

	public static void main(String[] args) throws InterruptedException {
		UnsafeCounter unsafeCounter = new UnsafeCounter();

		Runnable runnable = () -> {
			for (int i = 0; i < 1000; i++) {
				try {
					unsafeCounter.service();
				} catch (Exception e) {
				}
			}
		};

		new Thread(runnable).start();
		new Thread(runnable).start();

		sleep(1000);

		System.out.println(unsafeCounter.getCount());
	}
}
```

# Atomicity
To avoid race conditions, there must be a way to prevent other threads from using a variable while we're in the middle of modifying it, so we can ensure that other threads can observe or modify the state only before we start or after we finish, but not in the middle.

Operations A and B are atomic with respect to each other if, from the perspective of a thread executing A, when another thread executes B, either all of B has executed or none of it has.

An atomic operation is one that is atomic with respect to all operations, including itself, that operate on the same state.

To ensure thread safety, check-then-act operations and read-modify-write operations must always be atomic.

Sequences of operations that must be executed atomically in order to remain thread-safe are called compound actions.

## atomicity with one state
The **java.util.concurrent.atomic** package contains atomic variable classes for effecting atomic state transitions on numbers and object references.

```
@ThreadSafe
public class SafeCounter {
	private final AtomicLong count = new AtomicLong(0);

	public long getCount() {
		return count.get();
	}

	public void service() {
		count.incrementAndGet();
	}

	public static void main(String[] args) throws InterruptedException {
		SafeCounter safeCounter = new SafeCounter();

		Runnable runnable = () -> {
			for (int i = 0; i < 1000; i++) {
				try {
					safeCounter.service();
				} catch (Exception e) {
				}
			}
		};

		new Thread(runnable).start();
		new Thread(runnable).start();

		sleep(1000);

		System.out.println(safeCounter.getCount());
	}
}
```

Where practical, use existing thread-safe objects, like AtomicLong, to manage your class's state. It is simpler to reason about the possible states and state transitions for existing thread-safe objects than it is for arbitrary state variables, and this makes it easier to maintain and verify thread safety.

## atomicity with more than one states
只有一个state时，我们可以使用AtomicXXX来保证thread-safe，那么如果是多个states呢？每个state都使用AtomicXXX能保证thread-safe吗？不能。

```
@NotThreadSafe
public class UnsafeCacher {
	private AtomicLong lastReq = new AtomicLong();
	private AtomicLong lastRes = new AtomicLong();

	public Long service(Long req) {
		if (req.equals(lastReq.get())) {
			return lastRes.get();
		} else {
			Long res = req * 2;
			lastReq.set(req);
			lastRes.set(res);
			return res;
		}
	}

	public static void main(String[] args) throws InterruptedException {
		UnsafeCacher unsafeCacher = new UnsafeCacher();

		Runnable runnable = () -> {
			for (int i = 0; i < 10000; i++) {
				try {
					long res = unsafeCacher.service(Long.valueOf(i));
					if (res != i * 2) {
						System.out.print(i + ":" + res);
					}
				} catch (Exception e) {
				}
			}
		};

		new Thread(runnable).start();
		new Thread(runnable).start();

		sleep(1000);
	}
}
```

这里的问题在于虽然每个atomic variable的操作是原子的，但多个states的compound actions不是原子的。

To preserve state consistency, update related state variables in a single atomic operation.

Java provides a built-in locking mechanism for enforcing atomicity: the synchronized block. 

A synchronized block has two parts, a reference to an object that will serve as the lock, and a block of code to be guarded by that lock.

```
@ThreadSafe
public class SafeCacher {
	@GuardedBy("this")
	private AtomicLong lastReq = new AtomicLong();

	@GuardedBy("this")
	private AtomicLong lastRes = new AtomicLong();

	public synchronized Long service(Long req) {
		if (req.equals(lastReq.get())) {
			return lastRes.get();
		} else {
			Long res = req * 2;
			lastReq.set(req);
			lastRes.set(res);
			return res;
		}
	}

	public static void main(String[] args) throws InterruptedException {
		SafeCacher safeCacher = new SafeCacher();

		Runnable runnable = () -> {
			for (int i = 0; i < 10000; i++) {
				try {
					long res = safeCacher.service(Long.valueOf(i));
					if (res != i * 2) {
						System.out.print(i + ":" + res);
					}
				} catch (Exception e) {
				}
			}
		};

		new Thread(runnable).start();
		new Thread(runnable).start();

		sleep(1000);
	}
}
```

# Intrinsic locks
Every Java object can implicitly act as a lock for purposes of synchronization, these built-in locks are called *intrinsic locks* or *monitor locks*.

The lock is automatically acquired by the executing thread before entering a **synchronizaed** block and automatically released when control exits the **synchronizaed** block, whether by the normal control path or by throwing an exception out of the block.

Intrinsic locks in Java act as *mutexes* (or *mutual exclusion locks*), which means that at most one thread may own the lock. When thread A attempts to acquire a lock held by thread B, A must wait, or block, until B releases it. If B never releases the lock, A waits forever.

Since only one thread at a time can execute a block of code guarded by a given lock, the **synchronized** blocks guarded by the same lock execute atomically with respect to one another.

There is no inherent relationship between an object's intrinsic lock and its state, an object's fields need not be guarded by its intrinsic lock, though this is a perfectly valid locking convention that is used by many classes.

## reentrancy
Java intrinsic locks are *reentrant*, if a thread tries to acquire a lock that it already holds, the request succeeds.

Reentrancy means that locks are acquired on a per-thread rather than per-invocation basis.

Reentrancy is implemented by associating with each lock an acquisition count and an owning thread. When the count is zero, the lock is considered unheld. When a thread acquires a previously unheld lock, the JVM records the owner and sets the acquisition count to one. If that same thread acquires the lock again, the count is incremented, and when the owing thread exits the **synchronized** block, the count is decremented. When the count reaches zero, the lock is released.

Reentrancy facilities encapsulation of locking behavior, and thus simplifies the development of object-oriented concurrent code.

```
public class Widget {
    public synchronized void doSomething() {
    }
}

public class LoggingWidget extends Widget {
    public synchronized void doSomething() {
        System.out.println(toString() + ": calling doSomething");
        super.doSomething();
    }  
}
```

Without reentrant locks, the above code would be dead lock. Bacause the **doSomething** methods in **Widget** and **LoggingWidget** are both **synchronized**, each tries to acquire the lock on the **Widget** before proceeding. But if intrinsic locks were not reentrant, the call to **super.doSomething** would never be able to acquire the lock because it would be considered already held, and the thread would permanently stall waiting for a lock it can never acquire. Reentrancy saves us from deadlock in situations like this.

# Locking practices
If synchronization is used to coordinate access to a variable, it is needed *everywhere that variable is accessed*. When using locks to coordinate access to a variable, the same lock must be used wherever that variable is accessed.

It is a common mistake to assume that synchronization needs to be used only when *writing* to shared variables; *this is simply not true*.

For each mutable state variable that may be accessed by more than one thread, all accesses to that variable must be performed with the same lock held. In this case, we say that the variable is *guarded by* that lock.

Every shared, mutable variable should be guarded by exactly one lock. Make it clear to maintainers which lock that is.

A common locking convention is to encapsulate all mutable state within an object and to protect it from concurrent access by synchronizing any code path that accesses mutable state using the object's intrinsic lock. This pattern is used by many thread-safe classes, such as **Vector** and other synchronized collection classes.

When a class has invariants that involve more than one state variable, there is an additional requirement: each variable participating in the invariant must be guarded by the *same* lock.

For every invariant that involves more than one variable, all the variables involved in that invariant must be guarded by the same lock.

If synchronization is the cure for race conditions, why not just declare every method **synchronized**? While synchronized methods can make individual operations atomic, additional locking is required when multiple operations are combined into a compound action. At the same time, synchronizing every method can lead to liveness or performance problems.

# Liveness and performance
What's the problem with **SafeCacher**? It exhibits poor concurrency: when multiple requests arrive, they queue up and are handled sequentially. You should be careful not to make the scope of the **synchronized** block too small; you would not want to divide an operation that should be atomic into more than one **synchronized** block. But it is reasonable to try to exclude from **synchronized** blocks long-running operations that do not affect shared state, so that other threads are not prevented from accessing the shared state while the long-running operation is in progress.

```
@ThreadSafe
public class ImprovedSafeCacher {
	@GuardedBy("this")
	private Long lastReq;

	@GuardedBy("this")
	private Long lastRes;

	@GuardedBy("this")
	private Long hits;

	@GuardedBy("this")
	private Long cacheHits;

	public Long service(Long req) {
		Long res = null;
		synchronized (this) {
			++hits;
			if (req.equals(lastReq)) {
				++cacheHits;
				res = lastRes;
			}
		}

		if (res == null) {
			res = req * 2;
			synchronized (this) {
				lastReq = req;
				lastRes = res;
			}
		}

		return res;
	}

	public static void main(String[] args) throws InterruptedException {
		ImprovedSafeCacher safeCacher = new ImprovedSafeCacher();

		Runnable runnable = () -> {
			for (int i = 0; i < 10000; i++) {
				try {
					long res = safeCacher.service(Long.valueOf(i));
					if (res != i * 2) {
						System.out.print(i + ":" + res);
					}
				} catch (Exception e) {
				}
			}
		};

		new Thread(runnable).start();
		new Thread(runnable).start();

		sleep(1000);
	}
}
```

Deciding how big or small to make **synchronized** blocks may require tradeoffs among competing design forces, including safety, simplicity, and performance.

There is frequently a tension between simplicity and performance. When implementing a synchronization policy, resist the temptation to prematurely sacrifice simplicity (potentially compromising safety) for the sake of performance.

Whenever you use locking, you should be aware of what the code in the block is doing and how likely it is to take a long time to execute. Holding a lock for a long time, either because you are doing something compute-intensive or because you execute a potentially blocking operation, introduces the risk of liveness or performance problems. Avoid holding locks during lengthy computations or operations at risk of not completing quickly such as network or console I/O.