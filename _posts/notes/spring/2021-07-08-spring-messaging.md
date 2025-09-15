# Asynchronous Messaging
asynchronous messaging is a way of indirectly sending messages from one application to another without waiting for a response.

asynchronous messaging affords looser coupling and greater scalability between the communicating applications.

Spring提供了三种asynchronous messaging:
- JMS (Java Message Service)
- RabbitMQ and AMQP (Advanced Message Queueing Protocols)
- Apache Kafka

## pull mode vs push mode
- pull mode: your code requests a message and waits until one arrives
- push mode: messages are handed to your code as they become available

Both options are suitable for a variety of use cases. It is generally accepted that the push model is the best choice, as it doesn't block a thread. But in some use cases, a listener could be overburdened if messages arrive too quickly. The pull model enables a consumer to declare that they're ready to process a new message. When the message handlers need to be able to ask for more messages on their own timing, the pull model is more fitting.

# JMS
JMS is a Java standard that defines a common API for working with message brokers. 就像JDBC是操作关系数据库的统一接口一样，JMS提供了操作message brokers的统一接口。

## Spring JMS配置
添加Apache ActiveMQ Artemis dependency:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-artemis</artifactId>
</dependency>
```

通过配置spring.artemis.xxx，你就可以配置Artemis  broker相关信息：
```properties
spring.artemis.host
spring.artemis.port
spring.artemis.user
spring.artemis.password
```

With a JMS starter dependency in your build, Spring Boot will autoconfigure a JmsTemplate that you can inject and use to send and receive messages.

## JmsTemplate
就像Spring JDBC提供了JdbcTemplate一样，Spring JMS提供了JmsTemplate. Using JmsTemplate, it is easy to send messages across queues and topics from the producer side and to receive those messages on the consumer side.

如果没有JmsTemplate，那么you'd need to write code to create a connection and session with the message broker, and more code to deal with any exceptions that might be thrown in the course of sending a message.

Spring还支持message-driven POJOs, message-driven POJOs就是simple Java objects that react to messags arriving on a queue or topic in an asynchronous fashion.

### 发送消息
JmsTemplate提供了以下发送消息的methods，选择合适的使用即可：
- void send(MessageCreator)
- void send(Destination, MessageCreator)
- void send(String, MessageCreator)
- void convertAndSend(Object)
- void convertAndSend(Destination, Object)
- void convertAndSend(String, Object)
- void convertAndSend(Object, MessagePostProcessor)
- void convertAndSend(Destination, Object, MessagePostProcessor)
- void convertAndSend(String, Object, MessagePostProcessor)

如果你使用的方法没有地址参数，那么你得配置spring.jms.template.default-destination.

convertAndSend是如何convert的呢？It is achieved with an implementation of MessageConverter that does the dirty work of converting objects to Messages. 需要你实现MessageConverter吗？大部分情况下不需要，Spring JMS提供了几种实现，选择合适的定义成bean即可
- MappingJackson2MessageConverter, uses the Jackson 2 JSON library to convert messages to and from JSON
- MarshallingMessageConverter, uses JAXB to convert messages to and from XML
- MessagingMessageConverter
- SimpleMessageConverter

MessagePostProcessor有什么用？使用send()，你可以自己控制如何转化成messages；使用convertAndSend()的话，如果你需要定制化某些东西，就可以通过MessagePostProcessor来实现。

### 接收消息
JmsTemplate支持pull mode接收消息：
- receive()
- receive(Destination)
- receive(String)
- receiveAndConvert()
- receiveAndConvert(Destination)
- receiveAndConvert(String)

## @JmsListener
JMS也支持push mode接收消息，使用@JmsListener即可：
```java
public class JmsOrderListener {
	@JmsListener(destination = "tacocloud.order.queue")
	public void receiveOrder(Order order) {
		log.info(order.toString());
	}
}
```

# AMQP(Advanced Message Queueing Protocols)
相比JMS，AMQP使用了更高级的message-routing strategy. JMS messages are addressed with the name of a destination from which the receiver will retrieve them. AMQP messages are addressed with the name of an exchange and a routing key, which are decoupled from the queue that the receiver is listening to.

# RabbitMQ
RabbitMQ是one of the most prominent implementation of AMQP.

RabbitMQ发送消息流程：
- 首先message到exchange，when a message arrives at the RabbitMQ broker, it goes to the exchange for which it was addressed
- 然后根据exchange  type、binding between the exchange and queues、message routing key等决定路由到哪些queues, the exchange is responsible for routing it to one or more queues, depending on the type of exchange, the binding between the exchange and queues, and the value of the message's routing key

## exchange types
- **default**: a special exchange that's automatically created by the broker, it routes messages to queues whose name is the same as the message's routing key, all queues will automatically be bound to the default exchange
- **direct**: route messages to a queue whose binding key is the same as the message's routing key
- **topic**: route messages to one or more queues where the binding key (which may contain wildcards) matches the message's routing key
- **fanout**: route messages to all bound queues without regrad for binding keys or routing keys
- **headers**: similar to topic exchange, except the routing is based on message header values rather than routing keys
- **dead letter**: a catch-all for any message that are undeliverable (meaning they don't match any defined exchange-to-queue binding)

The simplest forms of exchanges are **default** and **fanout**, as these roughly correspond to a JMS queue and topic.

The most important thing to understand is that messages are sent to exchanges with routing keys and they are consumed from queues. How they get from an exchange to a queue depends on the binding definitions and what best suits your use cases.

Which exchange type you use and how you define the bindings from exchanges to queues has little bearing on how messages are sent and received in your Spring application.

## Spring RabbitMQ配置
添加Spring Boot AMQP starter dependency:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```
Adding the AMQP starter to your build will trigger autoconfiguration that will create a AMQP connection factory and RabbitTemplate beans, as well as other supporting components. Simply adding this dependency is all you need to do to start sending and receiving messages from a RabbitMQ broker with Spring.

通过配置spring.rabbitmq.xxx，你就可以配置RabbitMQ broker相关信息：
```properties
spring.rabbitmq.addresses
spring.rabbitmq.host
spring.rabbitmq.port
spring.rabbitmq.username
spring.rabbitmq.password
spring.rabbitmq.template.exchange
spring.rabbitmq.template.routing-key
spring.rabbitmq.template.receive-timeout
```

## RabbitTemplate
### 发送消息
RabbitTemplate，和JmsTemplate类似，提供了很多发送消息的methods:
- void send(message)
- void send(routingKey, message)
- void send(exchange, routingKey, message)
- void convertAndSend(message)
- void convertAndSend(routingKey, message)
- void convertAndSend(exchange, routingKey, message)
- void convertAndSend(message, messagePostProcessor)
- void convertAndSend(routingKey, message, messagePostProcessor)
- void convertAndSend(exchange, routingKey, message, messagePostProcessor)

和JmsTemplate最大的区别就是路由参数的不同，RabbitTemplate通过exchange和routing key来路由，缺省exchange就路由到default exchange，缺省routing key的会使用default routing key.

注意，default exchange和default routing key的名字都是""，你可以通过spring.rabbitmq.template.exchange和spring.rabbitmq.template.routing-key来自定义。

convertAndSend是如何convert的呢？It is achieved with an implementation of MessageConverter that does the dirty work of converting objects to Messages. 需要你实现MessageConverter吗？大部分情况下不需要，Spring AMQP提供了几种实现，选择合适的定义成bean即可：
- Jackson2JsonMessageConverter: converts object to and from JSON using the Jackson 2 JSON processor
- MarshallingMessageConverter: converts using a Spring Marshaller and Unmarshaller
- SerializerMessageConverter: converts String and native objects of any kind using Spring's Serializer and Deserializer abstractions
-  SimpleMessageConverter: converts String, byte arrays, and Serializable types

### 接收消息
RabbitTemplate提供了以下常用的接收消息方法：
- Message receive()
- Message receive(queueName)
- Message receive(timeoutMillis)
- Message receive(queueName, timeoutMillis)
- Object receiveAndConvert()
- Object receiveAndConvert(queueName)
- Object receiveAndConvert(timeoutMillis)
- Object receiveAndConvert(queueName, timeoutMillis)
- \<T\> T receiveAndConvert(ParameterizedTypeReference\<T\> type)
- \<T\> T receiveAndConvert(queueName, ParameterizedTypeReference\<T\> type)
- \<T\> T receiveAndConvert(timeoutMillis, ParameterizedTypeReference\<T\> type)
- \<T\> T receiveAndConvert(queueName, timeoutMillis, ParameterizedTypeReference\<T\> type)

这些接收消息的方法与发送消息最大的不同就是接收消息不涉及exchange和routing key，因为exchange和routing key是用来路由消息到queues，接收消息只需要知道使用哪个queue就可以了。

## @RabbitListener
使用@RabbitListener修饰对应的方法，这样当有消息到达队列时，就会执行这个方法。
```java
public class RabbitmqOrderListener {

    @Autowired
    private FileWriterGateway fileWriterGateway;

    @RabbitListener(queues = "tacocloud.order")
    public void receiveOrder(Order order) {
        log.info("order: " + order);
        fileWriterGateway.writeToFile("tacocloud.order", JSON.toJSONString(order));
    }
}
```

# Kafka
## Spring Kafka配置
添加Kafka dependency:
```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```
This will trigger Spring Boot autoconfiguration for Kafka that will arrange for a KafkaTemplate in the Spring application context.

通过配置spring.kafka.xxx，你就可以配置Kafka broker相关信息：
```properties
spring.kafka.bootstrap-servers
spring.kakfa.template.default-topic
spring.kafka.consumer.group-id
spring.kakfa.consumer.auto-offset-reset
```

## KafkaTemplate
KafkaTemplate提供了以下发送消息的方法：
- ListenableFuture<SendResult<K, V>> send(topic, data)
- ListenableFuture<SendResult<K, V>> send(topic, key, data)
- ListenableFuture<SendResult<K, V>> send(topic, partition, key, data)
- ListenableFuture<SendResult<K, V>> send(topic, partition, timestamp, key, data)
- ListenableFuture<SendResult<K, V>> sendDefault(data)
- ListenableFuture<SendResult<K, V>> sendDefault(key, data)
- ListenableFuture<SendResult<K, V>> sendDefault(partition, key, data)
- ListenableFuture<SendResult<K, V>> sendDefault(partition, timestamp, key, data)

相比JmsTemplate和RabbitTemplate，KafkaTemplate is typed with generics and is able to deal with domain types directly when sending messages，所以KafkaTemplate不需要convertAndSend().

相比JmsTemplate和RabbitTemplate，传参也有较大不同：
- topic必传，表示the topic to send the message to
- data必传，表示要传输的message
- partition可选，表示the partition to write the topic to
- key可选
- timestamp可选，默认取System.currentTimeMillis()

相比JmsTemplate和RabbitTemplate，KafkaTemplate没有接收消息的方法，the only way to consume messages from a Kafka topic is to write a message listener.

## @KafkaListener
使用@KafkaListener修饰对应的方法即可，与@JmsListener和@RabbitListener类似。