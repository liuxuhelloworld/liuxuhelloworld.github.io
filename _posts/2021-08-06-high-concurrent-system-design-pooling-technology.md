
# 什么是池化技术？
池化技术的核心思想是空间换时间，期望使用预先创建好的对象来减少频繁创建对象的开销，同时可以对对象进行统一的管理，降低了对象的使用成本。

# 数据库连接池
两个重要参数：最小连接数和最大连接数。建议最小连接数控制在10左右，最大连接数控制在20～30左右。

连接过程：
1. 如果当前连接数小于最小连接数，则创建新的连接处理请求
2. 如果连接池中有空闲连接，则复用空闲连接
3. 如果连接池中没有空闲连接，并且当前连接数小于最大连接数，则创建新的连接处理请求
4. 如果当前连接数已经大于等于最大连接数，则按照设定的等待时间等待可用连接
5. 如果等待超时，则抛出异常

连接池的管理维护：
最基本的问题就是如何保证连接池中的连接是有效的、可用的？一种方式是启动一个线程定期检测连接池中的连接是否可用。还有一种方式是获取到连接后先校验连接是否可用，这种方式在获取连接时引入了多余的开销，线上系统最好不要采取这种策略。

# 线程池
## ThreadPoolExecutor
**ThreadPoolExecutor**是JDK 1.5引入的线程池实现。类似的，有两个重要参数，核心线程数和最大线程数。

连接过程：
1. 如果线程池中的线程数小于核心线程数，则创建新的线程处理任务；
2. 如果线程数大于核心线程数，则把任务丢到一个队列里，等待空闲的线程执行；
3. 当队列中的任务堆积满了的时候，则继续创建线程，直到达到最大线程数；
4. 当线程数达到最大线程数时，默认丢弃任务。

当线程数达到核心线程数时，JDK实现的线程池会把新任务放到一个等待队列里，而不是直接继续创建新线程。这种方式适用于CPU密集型任务，不适用于IO密集型任务。Tomcat使用的线程池就没有使用等待队列。

# 池化注意事项
- 池子的最大值和最小值的设置很重要，初期可以依据经验来设置，后面还是需要根据实际运行情况做调整
- 池子中的对象需要在使用之前预先初始化完成，这叫做池子的预热，比方说使用线程池时就需要预先初始化所有的核心线程。如果池子未经过预热可能会导致系统重启后产生比较多的慢请求
- 池化技术核心是一种空间换时间优化方法的实践，所以要关注空间占用情况，避免出现空间过度使用出现内存泄露或者频繁垃圾回收等问题