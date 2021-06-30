Most concurrent applications are organized around the execution of *tasks*: abstract, discrete units of work.

The first step in organizing a program around task execution is identifying sensible *task boundaries*. Ideally, tasks are *indenpdent* activities: work that doesn't depend on the state, result, or side effects of other tasks.

Server applications should exhibit both *good throughput* and *good responsiveness* under normal load. Further, applications should exhibit *graceful degradation* as then become overloaded, rather than simply falling over under heavy load. Choosing good task boundaries, coupled with a sensible *task execution policy* can help achieve these goals.

Most server applications offer a natural choice of task boundary: individual client requests. Web servers, mail servers, file servers, EJB containers, and database servers all accept requests via network connections from remote clients. Using individual requests as task boundaries usually offer both independence and appropriate task sizing.

# Executing tasks in threads
## executing tasks sequentially
The simplest is to execute tasks sequentially in a single thread.

```java
public class SingleThreadWebServer {
	public static void main(String[] args) throws IOException {
		ServerSocket socket = new ServerSocket(80);
		while (true) {
			Socket connection = socket.accept();
			handleRequest(connection);
		}
	}

	private static void handleRequest(Socket connection) {
	}
}
```

Processing a web request involves a mix of computation and I/O. The server must perform socket I/O to read the request and write the response, which can block due to network congestion or connectivity problems. It may also perform file I/O or make database requests, which can also block. In a single-threaded server, blocking not only delays completing the current request, but prevents pending requests from being processed at all. If one request blocks for an unusually long time, users might think the server is unavailable because it appears unresponsive. At the same time, resource utilization is poor, since the CPU sits idle while the single thread waits for its I/O to complete.

In server applications, sequential processing rarely provides either good throughput or good responsiveness. In some situations, sequential processing may offer a simplicity or safety advantage; most GUI frameworks process tasks sequentially using a single thread.

## executing tasks concurrently
A more responsive approach is to create a new thread for servicing each request.

```java
public class ThreadPerTaskWebServer {
	public static void main(String[] args) throws IOException {
		ServerSocket serverSocket = new ServerSocket(80);
		while (true) {
			final Socket connection = serverSocket.accept();
			Runnable task = () -> handleRequest(connection);
			new Thread(task).start();
		}
	}

	private static void handleRequest(Socket connection) {
	}
}
```

Under light to moderate load, the thread-per-task approach is an improvement over sequential execution. As long as the request arrival rate does not exceed the server's capacity to handle requests, this approach offers better responsiveness and throughput.

For production use, however, the thread-per-task approach has some practical drawbacks, especially when a large number of threads may be created. Up to a certain point, more threads can improve throughput, but beyond that point creating more threads just slows down your application. The way to stay out of danger is to place some bound on how many threads your application creates, and to test your application thoroughly to ensure that, even when this bound is reached, it does not run out of resources.

# Executor framework
Tasks are logical units of work, and threads are a mechanism by which tasks can run asynchronously. The primary abstraction for task execution in the Java class libraries is not **Thread**, but **Executor**.

**Executor** forms the basis for a flexible and powerful framework for asynchronous task execution that supports a wide variety of task execution policies. It provides a standard means of decoupling *task submission* from *task execution*, describing tasks with **Runnable**. The **Executor** implementations also provide lifecycle support and hooks for adding statistics gathering, application management, and monitoring. Using an **Executor** opens the door to all sorts of additional opportunities for tuning, management, monitoring, logging, error reporting, and other possibilities that would have been far more difficult to add without a task execution framework.

Tasks are usually finite: they have a clear starting point and they eventually terminate. The lifecycle of a task executed by and **Executor** has four phases: *created, submitted, started, and completed*. Since tasks can take a long time to run, we also want to be able to cancel a task. In the **Executor** framework, tasks that have been summitted but not yet started can always be cancelled, and tasks that have been started can sometimes be cancelled if they are responsive to interruption.

**Future** represents the lifecycle of a task and provides methods to test whether the task has completed or been cancelled, retrieve its result, and cancel the task.

The **submit** methods in **ExecutorService** all return a **Future**, so that you can submit a **Runnable** or **Callable** to an executor and get back a **Future** that can be used to retrieve the result or cancel the task. Submitting a **Runnable** or **Callable** to an **Executor** constitutes a safe publication of the **Runnable** or **Callable** from the submitting thread to the thread that will eventually execute the task. Similarly, setting the result value for a **Future** constitutes a safe publication of the result from the thread in which it was computed to any thread that retrieves it via **get**.

**Executor** is based on the producer-consumer pattern, where activities that submit tasks are the producers and the threads that execute tasks are the consumers. *Using an **Executor** is usually the easiest path to implementing a producer-consumer design in your application*.

## executing tasks using Executor
```java
public class ExecutorWebServer {
	private static final int NTHREADS = 100;
//	private static final Executor EXECUTOR = new ThreadPerTaskExecutor();
	private static final Executor EXECUTOR = Executors.newFixedThreadPool(NTHREADS);

	public static void main(String[] args) throws IOException {
		ServerSocket serverSocket = new ServerSocket(80);
		while (true) {
			final Socket connection = serverSocket.accept();
			Runnable task = () -> handleRequest(connection);
			EXECUTOR.execute(task);
		}
	}

	private static void handleRequest(Socket connection) {
	}

	private static class ThreadPerTaskExecutor implements Executor {
		@Override
		public void execute(Runnable command) {
			new Thread(command).start();
		}
	}

	private static class WithinThreadExecutor implements Executor {
		@Override
		public void execute(Runnable command) {
			command.run();
		}
	}
}
```

## execution policy
The value of decoupling submission from execution is that it lets you easily specify, and subsequently change without great difficulty, the *execution policy* for a given class of tasks.

An execution policy specifies these:
- in what thread will tasks be executed?
- in what order should tasks be executed (FIFO, LIFO, priority order)?
- how many tasks may execute concurrently?
- how many tasks may be queued pending execution?
- if a task has to be rejected because the system is overloaded, which task should be selected as the victim, and how should the application be notified?
- what actions should be taken before or after executing a task?

Execution policies are a resource management tool, and the optimal policy depends on the available computing resources and your quality-of-service requirements. Separating the specification of execution policy from task submission makes it practical to select an execution policy at deployment time that is matched to the available hardware.

Whenever you see code of the form: **new Thread(runnable).start()** and you think you might at some point want a more flexible execution policy, seriously consider replacing it with the use of an **Executor**.

## thread pool
A thread pool manages a homogeneous pool of worker threads. A thread pool is tightly bound to a *work queue* holding tasks waiting to be executed. Worker threads have a simple life: request the next task from the work queue, execut it, and go back to waiting for another task.

Reusing an existing thread instead of creating a new one amortizes thread creation and teardown costs over multiple requests. As an added bonus, since the worker thread often already exists at the time the request arrives, the latency associated with thread creation does not delay task execution, thus improving responsiveness.

### Executors.newFixedThreadPool
A fixed-size thread pool creates threads as tasks are submitted, up to the maximum pool size, and then attempts to keep the pool size constant (adding new threads if a thread dies due to an unexpected **Exception**).

### Executors.newCachedThreadPool
A cached thread pool has more flexibility to reap idle threads when the current size of the pool exceeds the demand for processing, and to add new threads when demand increases, but places no bounds on the size of the pool.

### Executors.newSingleThreadExecutor
A single-threaded executor creates a single worker thread to process tasks, replacing it if it dies unexpectedly. Tasks are guaranteed to be processed sequentially according to the order imposed by the task queue (FIFO, LIFO, priority order).

### Executors.newScheduledThreadPool
A fixed-size thread pool that supports delayed and periodic task execution, similar to **Timer**. 

A **Timer** creates only a single thread for executing timer tasks. If a timer task takes too long to run, the timing accuracy for other **TimerTasks** can suffer. Scheduled thread pools address this limitation by letting you provide multiple threads for executing deferred and periodic tasks.

Another problem with **Timer** is that it behaves poorly if a **TimerTask** throws an unchecked exception. The **Timer** thread doesn't catch the exception, so an unchecked exception thrown from a **TimerTask** terminates the timer thread.

## executor shutdown 
In shutting down an application, there is a spectrum from graceful shutdown (finish what you've started but don't accept any new work) to abrupt shutdown (turn off the power to the machine room), and various points in between. 

The **ExecutorService** interface extends **Executor**, adding a number of methods for lifecycle management as well as some convenience methods for task submission.

The lifecycle implied by **ExecutorService** has three states, *running, shutting down, and terminated*. **ExecutorService** are initially created in the *running* state. The **shutdown** method initiates a graceful shutdown: no new tasks are accepted but previously submitted tasks are allowed to complete, including those that have not yet begun execution. The **shutdowNow** method initiates an abrupt shutdown: it attempts to cacel outstanding tasks and does not start any tasks that are queued but not begun. Once all tasks have completed, the **ExecutorService** transitions to the *terminated* state. You can wait for an **ExecutorService** to reach the terminated state with **awaitTermination**, or poll for whether it has yet terminated with **isTerminated**.

Tasks submitted to an **ExecutorService** after it has been shut down are handled by the *rejected execution handler*.

```java
public class LifecycleExecutorWebServer {
	private static final int NTHREADS = 100;
	private static final ExecutorService EXECUTOR = Executors.newFixedThreadPool(NTHREADS);

	public static void main(String[] args) throws IOException {
		ServerSocket serverSocket = new ServerSocket(80);
		while (!EXECUTOR.isShutdown()) {
			try {
				final Socket connection = serverSocket.accept();
				Runnable task = () -> handleRequest(connection);
				EXECUTOR.execute(task);
			} catch (RejectedExecutionException e) {
				if (!EXECUTOR.isShutdown()) {
					System.out.println("task submission rejected");
				}
			}
		}
	}

	public void stop() {
		EXECUTOR.shutdown();
	}

	private static void handleRequest(Socket connection) {
	}
}
```

## result-bearing tasks
**Runnable** is a fairly limiting abstraction; **run** cannot return a value or throw checked exceptions.

## heterogeneous tasks vs homogeneous tasks
Obtaining significant performance improvements by trying to parallelize sequential heterogeneous tasks can be tricky.
- assigning a different type of task to each worker does not scale well
- tasks may have disparate sizes
- dividing a task among multiple workers always involves some amount of coordination overhead

Trying to increase concurrency by parallelizing heterogeneous activities can be a lot of work, and there is a limit to how much additional concurrency you can get out of it.

The real performance payoff of dividing a program's workload into tasks comes when there are a large number of independent, *homogeneous* tasks that can be processed concurrently.

## completion service
If you have a batch of computations to submit to an **Executor** and you want to retrieve their results as they become available, you could retain the **Future** associated with each task and repeatedly poll for completion by calling **get** with a timeout of zero. This is possible, but tedious. 

**CompletionService** combines the functionality of an **Executor** and a **BlockingQueue**. You can submit **Callable** tasks to it for execution and use the queue like methods **take** and **poll** to retrieve completed results, packaged as **Future**s, as they become available.

**ExecutorCompletionService** implements **CompletionService**, delegating the computation to an **Executor**.


# Page Renderer Examples
## sequential page renderer
The simplest approach is to process the HTML document sequentially. As text markup is encountered, render it into the image buffer; as image references are encountered, fetch the image over the network and draw it into the image buffer as well. This is likely to annoy the user, who may have to wait a long time before all the text is rendered.

A less annoying but still sequential approach involves rendering the text elements first, leaving rectangular placeholders for the images, and after completing the initial pass on the document, going back and downloading the images and drawing them into the associated placeholder.

```java
public class SingleThreadPageRenderer {
	public void renderPage(CharSequence source) {
		renderText(source);

		List<ImageData> imageData = new ArrayList<>();

		for (ImageInfo imageInfo : scanForImageInfo(source)) {
			imageData.add(imageInfo.downloadImage());
		}

		for (ImageData data : imageData) {
			renderImage(data);
		}
	}

	private void renderText(CharSequence source) {}

	private void renderImage(ImageData imageData) {
	}

	private List<ImageInfo> scanForImageInfo(CharSequence source) {
		return new ArrayList<>();
	}

	private static class ImageData {}

	private static class ImageInfo {
		public ImageData downloadImage() {
			return new ImageData();
		}
	}
}
```

## concurrent page renderer
```java
public class FuturePageRenderer {
	private final ExecutorService executorService = Executors.newCachedThreadPool();

	public void renderPage(CharSequence source) {
		final List<ImageInfo> imageInfos = scanForImageInfo(source);
		Callable<List<ImageData>> task = new Callable<List<ImageData>>() {
			@Override
			public List<ImageData> call() throws Exception {
				List<ImageData> result = new ArrayList<>();
				for (ImageInfo imageInfo : imageInfos) {
					result.add(imageInfo.downloadImage());
				}
				return result;
			}
		};

		Future<List<ImageData>> future = executorService.submit(task);

		renderText(source);

		try {
			List<ImageData> imageData = future.get();

			for (ImageData data : imageData) {
				renderImage(data);
			}
		} catch (InterruptedException e) {
			Thread.currentThread().interrupt();

			future.cancel(true);
		} catch (ExecutionException e) {
			e.printStackTrace();
		}
	}

	private void renderText(CharSequence source) {}

	private void renderImage(ImageData imageData) {
	}

	private List<ImageInfo> scanForImageInfo(CharSequence source) {
		return new ArrayList<>();
	}

	private static class ImageData {}

	private static class ImageInfo {
		public ImageData downloadImage() {
			return new ImageData();
		}
	}
}
```
This allows the text to be rendered concurrently with downloading the image data. When all the images are downloaded, they are rendered onto the page. This is an improvement in that the user sees a result quickly and it exploit some parallelism, but we can do considerably better. There is no need for users to wait for all the images to be downloaded; they would probably prefer to see individual images drawn as they become available.