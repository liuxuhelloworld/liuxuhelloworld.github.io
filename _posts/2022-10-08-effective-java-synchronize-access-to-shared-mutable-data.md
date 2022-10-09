The **synchronized** keyword ensures that only a single thread can execute a method or block at one time.

Many programmers think of synchronization solely as a means of *mutual exclusion*, to prevent an object from being seen in an inconsistent state by one thread while it is being modified by another. This view is correct, but it is only half the story. Not only does synchronization prevent threads from observing an object in an inconsistent state, but it ensures that each thread entering a synchronized method or block sees the effects of all previous modifications that were guarded by the same lock. Without synchronization, one thread's changes might not be visible to other threads.

The language specification guarantees that reading or writing a variable is *atomic* unless the variable is of type **long** or **double**. You may hear it said that to improve performance, you should dispense with synchronization when reading or writing atomic data. This advice is dangerously wrong. While the language specification guarantees that a thread will not see an arbitrary value when reading a field, it does not guarantee that a value written by one thread will be visible to another.

Synchronization is required for reliable communication between threads as well as for mutual exclusion.

A recommended way to stop one thread from another is to have the first thread poll a **boolean** field that is initially **false** but can be set to **true** by the second thread to indicate that the first thread is to stop itself.

```java
public class StopThreadBad {
	private static boolean stopRequested;

	public static void main(String[] args) throws InterruptedException {
		Thread backgroundThread = new Thread(() -> {
			int i = 0;
			while (!stopRequested) {
				i++;
			}
		});
		backgroundThread.start();

		TimeUnit.SECONDS.sleep(1);

		stopRequested = true;
	}
}
```

The above code doesn't work. The problem is that in the absence of synchronization, there is no guarantee as when, if ever, the background thread will see the change in the value of **stopRequested** made by the main thread. The result is a *liveness failure*: the program fails to make progress.

One way to fix the problem is to synchronize access to the **stopRequested** field.

```java
public class StopThreadGood {
	private static boolean stopRequested;

	private static synchronized void requestStop() {
		stopRequested = true;
	}

	private static synchronized boolean isStopRequested() {
		return stopRequested;
	}

	public static void main(String[] args) throws InterruptedException {
		Thread backgroundThread = new Thread(() -> {
			int i = 0;
			while (!isStopRequested()) {
				i++;
			}
		});
		backgroundThread.start();

		TimeUnit.SECONDS.sleep(1);

		requestStop();
	}
}
```

Note that both the write method and the read method are synchronized. It is not sufficient to synchronized only the write method. Synchronization is not guaranteed to work unless both read and write operations are synchronized.

The actions of the synchronized methods in **StopThreadGood** would be atomic even without synchronization. In other words, the synchronization on these methods is used solely for its communication effects, not for mutual exclusion. In this case, we can change to use **volatile**. While the **volatile** modifier performs no mutual exclusion, it guarantees that any thread that reads the field will see the most recently written value.

```java
public class StopThreadGoodUsingVolatile {
	private static volatile boolean stopRequested;

	public static void main(String[] args) throws InterruptedException {
		Thread backgroundThread = new Thread(() -> {
			int i = 0;
			while (!stopRequested) {
				i++;
			}
		});
		backgroundThread.start();

		TimeUnit.SECONDS.sleep(1);

		stopRequested = true;
	}
}
```

The best way to avoid the problem discussed in this item is not to share mutable data. Either share immutable data or don't share at all. In other words, confine mutable data to a single thread. If you adopt this policy, it is important to document it so that the policy is maintained as your program evolves. It is also important to have a deep understanding of the frameworks and libraries you're using because they may introduce threads that you are unaware of.

In summary, when multiple threads share mutable data, each thread that reads or writes the data must perform synchronization. In the absence of synchronization, there is no guarantee that one thread's changes will be visible to another thread. The penalties for failing to synchronize shared mutable data are liveness and safety failures. These failures are among the most difficult to debug. They can be intermittent and timing-dependent, and program behavior can vary radically from one VM to another. If you need only inter-thread communication, and not mutual exclusion, the **volatile** modifier is an acceptable form of synchronization, but it can be tricky to use correctly.