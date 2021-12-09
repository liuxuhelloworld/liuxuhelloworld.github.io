In particular, multithreaded programs require developers to address concerns that aren't issues for single-threaded programs, particularly issues of safety and liveness. Consequently, different threads have to be extremely careful about which resources they use when. Generally, each thread must agree to use certain resources only when it is sure those resources can't change or that it has exclusive access to them. However, it's also possible for two threads to be too careful, each waiting for exclusive access to resources it will never get. This can lead to deadlock, in which two threads are each waiting for resources the other possesses. Neither thread can proceed without the resources that the other thread has reserved, but neither is willing to give up the resources it has already.

# Thread and Runnable
```java
public class DigestThread extends Thread {
	private String fileName;

	public DigestThread(String fileName) {
		this.fileName = fileName;
	}

	@Override
	public void run() {
		try {
			FileInputStream in = new FileInputStream(fileName);
			MessageDigest sha = MessageDigest.getInstance("SHA-256");
			DigestInputStream din = new DigestInputStream(in, sha);

			while (din.read() != -1) {
			}

			din.close();

			byte[] digest = sha.digest();

			StringBuilder builder = new StringBuilder(fileName);
			builder.append(": ");
			builder.append(DatatypeConverter.printHexBinary(digest));

			System.out.println(builder);
		} catch (IOException e) {
			System.err.println(e);
		} catch (NoSuchAlgorithmException e) {
			System.err.println(e);
		}
	}

	public static void main(String[] args) {
		for (String fileName : args) {
			Thread t = new DigestThread(fileName);
			t.start();
		}
	}
}
```

```java
public class DigestRunnable implements Runnable {
	private String fileName;

	public DigestRunnable(String fileName) {
		this.fileName = fileName;
	}

	@Override
	public void run() {
		try {
			FileInputStream in = new FileInputStream(fileName);
			MessageDigest sha = MessageDigest.getInstance("SHA-256");
			DigestInputStream din = new DigestInputStream(in, sha);

			while (din.read() != -1) {
			}

			din.close();

			byte[] digest = sha.digest();

			StringBuilder builder = new StringBuilder(fileName);
			builder.append(": ");
			builder.append(DatatypeConverter.printHexBinary(digest));

			System.out.println(builder);
		} catch (IOException e) {
			System.err.println(e);
		} catch (NoSuchAlgorithmException e) {
			System.err.println(e);
		}
	}

	public static void main(String[] args) {
		for (String fileName : args) {
			DigestRunnable dr = new DigestRunnable(fileName);
			Thread t = new Thread(dr);
			t.start();
		}
	}
}
```

# Returning Information from a Thread
One of the hardest things for programmers accustomed to traditional, single-threaded procedural models to grasp when moving to a multithreaded environment is how to return information from a thread. Getting information out of a finished thread is one of the most commonly misunderstood aspects of multithreaded programming.

# Synchronization
Threads make programs more efficient by sharing memory, file handles, sockets, and other resources. As long as two threads don't want to use the same resource at the same time, a multithreaded program is much more efficient than the multiprocess alternative, in which each process has to keep its own copy of every resource. The downside of a multithreaded program is that if two threads want the same resource at the same time, one of them will have to wait for the other to finish. If one of them doesn't wait, the resource may be corrupted.

There are a number of techniques that avoid the need for synchronization entirely:
- use local variables instead of fields wherever possible
- use immutable class
- use a thread-unsafe class but only as a private field of a class that is thread safe

# Deadlock
Synchronization can lead to another possible problem: deadlock. Deadlock occurs when two threads need exclusive access to the same set of resources and each thread holds the lock on a different subset of those resources.

The most important technique for preventing deadlock is to avoid unnecessary synchronization. If there's an alternative approach for ensuring thread safety, such as making objects immutable or keeping a local copy of an object, use it. Synchronization should be a last resort for ensuring thread safety. If you do need to synchronize, keep the synchronized blocks small and try not to synchronize on more than one object at a time. This can be tricky, thought, because many of the methods from the Java class library that your code may invoke synchronize on objects you aren't aware of. Consequently, you may in fact be synchronizing on many objects than you expect.

If multiple objects need the same set of shared resources to operate, make sure they request them in the same order.

# Thread Scheduling
Each thread has a priority, specified as an integer from 0 to 10.

Every virtual machine has a thread scheduler that determines which thread to run at any given time. There are two kinds of thread scheduling: preemptive and cooperative. A preemptive thread scheduler determines when a thread has had its fair share of CPU time, pauses that thread, and then hands off control of the CPU to a different thread. A cooperative thread scheduler waits for the running thread to pause itself before handing off control of the CPU to a different thread. A virtual machine that uses cooperative thread scheduling is much more susceptible to thread starvation than a virtual machine that uses preemptive thread scheduling, because one high-priority, uncooperative thread can hog an entire CPU.

All Java virtual machines are guaranteed to use preemptive thread scheduling between priorities.

It is important to make sure all your threads periodically pause themselves so that other threads have an opportunity to run. There are many ways a thread can pause in favor of other threads or indicate that it is ready to pause:
- it can block on I/O
- it can block on a synchronized object
- it can yield
- it can go to sleep
- it can join another thread
- it can wait on an object
- it can finish

Inspect every **run()** method you write to make sure that one of these conditions will occur with reasonable frequency.

Blocking occurs any time a thread has to stop and wait for a resource it doesn't have. The most common way a thread in a network program will voluntarily give up control of the CPU is by blocking on I/O. Because CPUs are much faster than networks and disks, a network program will often block while waiting for data to arrive from the network or be sent out to the network. Even though it may block for only a few miliiseconds, this is enough time for other threads to do significant work. Threads can also block when they enter a synchronized method or block. If the thread does not already possess the lock for the object being synchronized on and some other thread does possess that lock, the thread will pause until the lock is released. Neither blocking on I/O nor blocking on a lock will release any locks the thread already possesses.

The second way for a thread to give up control is to explicitly yield. A thread does this by invoking the static **Thread.yield()** method. This signals to the virtual machine that it can run another thread if one is ready to run. Before yielding, a thread should make sure that it or its associated **Runnable** object is in a consistent state that can be used by other objects. Yielding does not release any locks the thread holds. Therefore, ideally, a thread should not be synchronized on anything when it yields.

Sleeping is more powerful form of yielding. Whereas yielding indicates only that a thread is willing to pause and let other equal-priority threads have a turn, a thread that goes to sleep will pause whether any other thread is ready to run or not. This gives an opportunity to run not only to other threads of the same priority, but also to threads of lower priorities. However, a thread that goes to sleep does hold onto all the locks it's grabbed. Consequently, other threads that need the same locks will be blocked even if the CPU is available. Therefore, try to avoid sleeping threads inside a synchronized method or block. It is also possible that some other thread will do something to wake up the sleeping thread before its time. Generally, this is accomplished by invoking the sleeping thread's **interrupt()** method, and the sleeping thread experiences as an **InterruptedException**.

It is not uncommon for one thread to need the result of another thread. A thread that's joined to another thread can be interrupted just like a sleeping thread if some other thread invokes its **interrupt()** method. Joining is perhaps not as important as it was prior to Java 5. In particular, many designs that used to require **join()** can now more easily be implemented using an **Executor** and a **Future** instead.

A thread can wait on an object it has locked. While waiting, it releases the lock on the object and pauses until it is notified by some other thread. Another thread changes the object in some way, notifies the thread waiting on the object, and then continues. This differs from joining in that neither the waiting nor the notifying thread has to finish before the other thread can continue. Waiting pauses execution until an object or resource reaches a certain state. Joining pauses execution until a thread finishes. 

To wait on a particular object, the thread that wants to pause must first obtain the lock on the object using **synchronized** and then invoke one of the object's three overloaded **wait()** methods. When one of the these methods is invoked, the thread that invoked it releases the lock on the object it's waiting on (though not any locks it possesses on other objects) and goes to sleep. It remains asleep until one of three things happens:
- the timeout expires
- the thread is interrupted
- the object is notified

Once a waiting thread is notified, it attempts to regain the lock of the object it was waiting on. If it succeeds, execution resumes with the statement immediately following the invocations of **wait()**. If it fails, it blocks on the object until its lock becomes available; the execution resumes with the statement immediately following the invocations of **wait()**.

The final way a thread can give up control of the CPU in an orderly fashion is by finishing. When the **run()** method returns, the thread dies and the other threads can take over. In network applications, this tends to occur with threads that wrap a single blocking operation, such as downloading a file from a server, so that the rest of the application won't be blocked. Otherwise, it your **run()** method is so simple that it always finishes quickly enough without blocking, there's a very real question of whether you should spawn a thread at all. There's a nontrivial amount of overhead for the virtual machine in setting up and tearing down threads. If a thread is finishing in a small fraction of a second anyway, chances are it would finish even faster if you used a simple method call rather than a separate thread.

# Thread Pools
The **Executors** class in **java.util.concurrent** makes it quite easy to set up thread pools. You simply submit each task as a **Runnable** object to the pool. You get back a **Future** object you can use to check on the progress of the task.