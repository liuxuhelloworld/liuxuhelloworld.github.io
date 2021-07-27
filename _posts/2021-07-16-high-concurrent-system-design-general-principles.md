# 横向扩展（Scale-out）
即分而治之，采用分布式集群的方式把流量分流开，让每个服务器承担一部分并发和流量。

与横向扩展对应的是纵向扩展（Scale-up），纵向扩展通过提高硬件配置来提升系统的并发处理能力。

比如，一个4核4G的系统现在能处理200QPS的流量，如果流量增大到400QPS呢？纵向扩展的思路是换一个8核8G性能更好的机器，横向扩展的思路是增加一台4核4G的机器组成一个集群来处理。

纵向扩展的问题在于受限于单机处理能力的极限。

横向扩展可以突破单机极限，但同时引入了一些复杂问题，比如：
- 不同节点信息的一致性
- 某些节点故障时整体的可用性
- 如何无感知的增加或删除节点

# 缓存
为什么使用缓存可以提高系统的并发处理能力呢？因为不同存储介质的访问速度差异非常大，缓存就是避免存取访问速度慢的存储介质。

可以参考[Latency Numbers Every Programmer Should Know](https://gist.github.com/jboner/2841832)

# 异步
与异步相对的是同步，那么什么是同步，什么是异步呢？

与同步和异步容易混淆在一起的概念是阻塞和非阻塞。

同步和异步指的是how the server will deal with incoming requests，阻塞和非阻塞指的是how the client will handle the results (waits or do something else)。

一种常见的误解是阻塞等价于同步，非阻塞等价于异步，但其实同步异步和阻塞非阻塞没有直接关系。

同步和异步描述的是通信机制(communication mechanism)：
- synchronous is, when we started a function call, the call will not return anything until it gets the result, the function needs to finish everything before it can give anything to us
- asynchronous does not need to wait for the function completes its operation, once we call it, it returns immediately without any result, the function uses callback function (or other notification method) to notify us to get the value after it completes the execution

阻塞和非阻塞描述的是the status of the program while waiting for the result from the function call:
- a blocking operation hangs up current thread before it gets the result, in other words, a blocking operation will let the current thread wait for the result returns, even if the target function will use a callback function to notify client side to fetch the result, the thread on the client side will still be blocked until it gets the returned result
- a non-blocking operation will not hang up the current thread if no result returned immediately

一个简单的例子说明同步异步、阻塞非阻塞概念，假设你打电话到一个酒店预定房间：
- 拨通电话以后，酒店前台查看是否能满足你的预定，并告知你结果，这就是同步
- 拨通电话以后，酒店前台确认收到你的预定申请了，等确认是否能预定有结果时回电告知你结果，这就是异步
- 在得到酒店前台的确认结果前，你就坐在电话旁等待，这就是阻塞
- 在告知酒店前台你的预定请求后，你就开始忙其他事情，这就是非阻塞