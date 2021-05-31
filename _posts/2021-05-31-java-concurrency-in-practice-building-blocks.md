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

