# 什么是Spring Integration?
Spring Integration is a ready-to-use implementation of many of the integration patterns that are catalogued in Enterprise Integration Patterns. Each pattern is implemented as a component through which messages ferry data in a pipeline. Using Spring configuration, you can assemble these components into a pipeline through which data flows. 

Spring Integration enables the creation of integration flows through which an application can receive or send data to some resource external to the application itself.

# 如何使用Spring Integration?
添加Spring Boot starter for Spring Integration以及对应的Spring Integration endpoint module，Spring Integration提供了很多针对不同对象的endpoint module.

```properties
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-integration</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.integration</groupId>
	<artifactId>spring-integration-file</artifactId>
</dependency>
```

然后配置integration flow:
- XML configuration, 略
- Java configuration, 配置integration flow涉及的各个bean，以及它们之间如何串联
```java
    @Bean
    public MessageChannel textInChannel() {
        return new DirectChannel();
    }

    @Bean
    public MessageChannel fileWriterChannel() {
        return new DirectChannel();
    }

    @Bean
    @Transformer(inputChannel = "textInChannel", outputChannel = "fileWriterChannel")
    public GenericTransformer<String, String> upperCaseTransformer() {
        return text -> text.toUpperCase();
    }

    @Bean
    @ServiceActivator(inputChannel = "fileWriterChannel")
    public FileWritingMessageHandler fileWriter() {
        FileWritingMessageHandler handler =
            new FileWritingMessageHandler(new File("./src/main/resources/spring_integration_test"));

        handler.setExpectReply(false);
        handler.setFileExistsMode(FileExistsMode.APPEND);
        handler.setAppendNewLine(true);

        return handler;
    }
```
- Java configuration with a DSL (Domain-Specific Language)，配置一个表示整个integration flow的bean
```java
    @Bean
    public IntegrationFlow fileWriterFlow() {
        return IntegrationFlows
            .from(MessageChannels.direct("textInChannel"))
            .<String, String>transform(t -> t.toUpperCase())
            .handle(Files
                .outboundAdapter(new File("./src/main/resources/spring_integration_test"))
                .fileExistsMode(FileExistsMode.APPEND)
                .appendNewLine(true))
            .get();
    }
```

# Spring Integration Endpoint Modules
Spring Integration provides over two dozen endpoint modules for integration with external systems: 
- AMQP: spring-integration-amqp
- Spring application events: spring-integration-event
- RSS and Atom: spring-integration-feed
- Filesystem: spring-integration-file
- FTP/FTPS: spring-integration-ftp
- GemFire: spring-integration-gemfire
- HTTP: spring-integration-http
- JDBC: spring-integration-jdbc
- JPA: spring-integration-jpa
- JMS: spring-integration-jms
- Email: spring-integration-mail
- MongoDB: spring-integration-mongodb
- MQTT: spring-integration-mqtt
- Redis: spring-integration-redis
- RMI: spring-integration-rmi
- SFTP: spring-integration-sftp
- STOMP: spring-integration-stomp
- Stream: spring-integration-stream
- Syslog: spring-integration-syslog
- TCP/UDP: spring-integration-ip
- Twitter: spring-integration-twitter
- Web services: spring-integration-ws
- WebFlux: spring-integration-webflux
- WebSocket: spring-integration-websocket
- XMPP: spring-integration-xmpp
- Zookeeper: spring-integration-zookeeper

# Integration Flow Components
- gateways: pass data into an integration flow via an interface
- channels: pass messages from one element to another
- channel adapters: connect a channel to some external system or transport. Can either accept input or write to the external system
- filters: conditionally allow messages to pass through the flow based on some criteria
- transformers: change message values or convert message payloads from one type to another
- routers: direct messages to one of serveral channels, typically based on message headers
- splitters: split incoming messages into two or more messages, each sent to different channels
- aggregators: the opposite of splitters, combining multiple messages coming in from separate channels into a single message
- service activators: hand a message off to some Java method for processing, and then publish the return value on an output channel

## gateways
Gateways are the means by which an application can submit data into an integration flow and, optionally, receive a response that's the result of the flow. Implemented by Spring Integration, gateways are realized as interfaces that the application can call to send messages to the integration flow.

```java
@Component
@MessagingGateway(defaultRequestChannel="inChannel", defaultReplyChannel="outChannel")
public interface UpperCaseGateway {
		String uppercase(String in);
}
```
When *uppercase()* is called, the given string is published to the integration flow into the channel nemed *inChannel*. And, regargless of how the flow is defined or what it does, when data arrives in the channel name *outChannel*, it is returned from the *uppercase()* method.

## channels
Channels are the means by which message move through an integration pipeline. They are the pipes that connect all the other parts of Spring Integration plumbing together.

Spring Integration provides several channel implementations:
- **PublishSubscribeChannel**: messages published into a **PublishSubscribeChannel** are passed on to one or more consumers. If there are multiple consumers, all of them receive the message
-  **DirectChannel**: like **PublishSubscribeChannel** but sends a message to a single consumer by invoking the consumer in the same thread as the sender. This allows for transactions to span across the channel
-  **ExecutorChannel**: similar to DirectChannel but the message dispatch occurs via a **TaskExecutor**, taking place in a separate thread from the sender. This channel type doesn't support transactions that can span the channel
- **QueueChannel**: messages published into a **QueueChannel** are stored in a queue until pulled by a consumer in a FIFO fashion. If there are multiple consumers, only one of them receives the message
- **PriorityChannel**: like QueueChannel but, rather than FIFO behavior, messages are pulled by consumers based on the message *prioprity* header
- **RendezvousChannel**: like QueueChannel except that the sender blocks the channel until a consumer receives the message, effectively synchronizing the sender with the consumer
- **FluxMessageChannel**: a Reactive Streams Publisher message channel based on Project Reactor's Flux

## channel adapters
Channel adapters represent the entry and exit points of an integration flow. Data enters an integration flow by way of an inbound channel adapter and exits an integration flow by way of an outbound channel adapter.

Inbound channel adapters can take many forms, depending on the source of the data they introduce into the flow.

```java
@Bean
@InboundChannelAdapter(poller=@Poller(fixedRate="1000"), channel="numberChannel")
public MessageSource<Integer> numberSource(AtomicInteger source) {
		return () -> {
				return new GenericMessage<>(source.getAndIncrement());
		};
}
```

Often, channel adapters are provided by one of Spring Integration's many endpoint modules. Spring Integration provides over two dozen endpoint modules containing channel adapters, both inbound and outbound, for integration with a variety of common external systems.

```java
@Bean
@InboundChannelAdapter(channel="file-channel", poller=@Poller(fixedDelay="1000"))
public MessageSource<File> fileReadingMessageSource() {
		FileReadingMessageSource sourceReader = new FileReadingMessageSource();
		sourceReader.setDirectory(new File(INPUT_DIR));
		sourceReader.setFilter(new SimplePatternFileListFilter(FILE_PATTERN));
		return sourceReader;
}
```

## filters
Filters can be placed in the midest of an integration pipeline to allow or disallow messages from proceeding to the next step in the flow.

```java
@Filter(inputChannel="numberChannel", outputChannel="evenNumberChannel")
public boolean evenNumberFilter(Integer number) {
		return number % 2 == 0;
}
```

## transformers
Transformers perform some operation on messages, typically resulting in a different message and, possibly, with a different payload type.

```java
@Bean
@Transformer(inputChannel="numberChannel", outputChannel="romanNumberChannel")
public GenericTransformer<Integer, String> romanNumTransformer() {
		return RomanNumbers::toRoman;
}
```

## routers
Routers, based on some routing criteria, allow for branching in an integration flow, directing messages to different channels.

```java
@Bean
public MessageChannel evenChannel() {
		return new DirectChannel();
}
@Bean
public MessageChannel oddChannel() {
		return new DirectChannel();
}
@Bean
@Router(inputChannel="numberChannel")
public AbstractMessageRouter evenOddRouter() {
		return new AbstractMessageRouter() {
				@Override
				protected Collection<MessageChannel> determineTargetChannels(Message<?> message) {
						Integer number = (Integer) message.getPayload();
						if (number % 2 == 0) {
								return Collections.singleton(evenChannel());
						}
						return Collections.singleton(oddChannel());
				}
		};
}
```

## splitters
Splitters break down messages into two or more separate messages that can be handled by separate subflows.

```java
public class OrderSplitter {
		public Collection<Object> splitOrderIntoParts(PurchaseOrder po) {
				ArrayList<Object> parts = new ArrayList<>();
				parts.add(po.getBillingInfo());
				parts.add(po.getLineItems());
				return parts;
		}
}

@Bean
@Splitter(inputChannel="poChannel", outputChannel="splitOrderChannel")
public OrderSplitter orderSplitter() {
		return new OrderSplitter();
}

@Bean
@Router(inputChannel="splitOrderChannel")
public MessageRouter splitOrderRouter() {
		PayloadTypeRouter router = new PayloadTypeRouter();
		router.setChannelMapping(BillingInfo.class.getName(), "billingInfoChannel");
		router.setChannelMapping(List.class.getName(), "lineItemsChannel");
		return router;
}

@Splitter(inputChannel="lineItemsChannel", outputChannel="lineItemChannel")
public List<LineItem> lineItemSplitter(List<LineItem> lineItems) {
		return lineItems;
}
```

## service activators
Service activators receive messages from an input channel and send those messages to an implementation of **MessageHandler**.

Service activators often serve the purpose of an outbound channal adapter, especially when data needs to be handed off to the application itself.

```java
@Bean
@ServiceActivator(inputChannel="someChannel")
public MessageHandler sysoutHandler() {
		return message -> {
				System.out.println("Message payload: " + message.getPayload());
		};
}

@Bean
@ServiceActivator(inputChannel="orderChannel", outputChannel="completeOrder")
public GenericHandler<Order> orderHandler(OrderRepository orderRepo) {
		return (payload, headers) -> {
				return orderRepo.save(payload);
		};
}
```