# 消息有哪些应用场景？
## 削峰
![削峰示意图](https://static001.geekbang.org/resource/image/de/ad/de0a7a65a0bf51e1463d40d666a034ad.jpg)

## 异步
![异步示意图](https://static001.geekbang.org/resource/image/3b/aa/3b19c4b5e93eeb32fd9665e330e6efaa.jpg)

## 解藕
![解藕示意图](https://static001.geekbang.org/resource/image/6e/f6/6e096e287f2c418f663ab201f435a5f6.jpg)

# 使用消息需要注意什么？
## 消息丢失
哪些场景下消息可能丢失呢？

生产者写消息队列时，比如网络抖动可能导致消息丢失，可以通过消息重传减少这种丢失。

消息队列内部存储出错，比如使用了异步刷盘机制，机器掉电或异常重启可能导致消息丢失，可以通过集群部署减少这种丢失。

## 消息重复
无论生产者还是消费者，都可以通过使用幂等id来保证幂等性。

## 消息延迟
## 消息乱序