# 什么是microservice？
Microservice is small, independent applications that work together to provide the functionality of a compelte application. It is a way of factoring an application into small-scale, miniature applications that are independently developed and deployed.

# 为什么要microservice?
与microservice对应的叫monolithic application: 
- monoliths are difficult to reason about
- monoliths are more difficult to test
- monoliths are more prone to library conflicts
- monoliths scale inefficiently
- technology decisions for a monolith are made for the entire monolith
- monoliths require a great deal of ceremony to get to production

对应的，microservice的好处包括：
- microservices can be easily understood
- microservices are easier to test
- microservices are unlikely to suffer from library incompatibilities
- microservices scale independently
- technology choices can be made differently for each microservice
- microservices can be published to production more frequently

注意，microservices不是免费午餐，not all applications require or benefit from microservice architecture. 如果应用本身就很小，monolith一样ok，一样可以结构清晰易维护，只有当应用足够复杂时，再考虑使用microservice架构。

# Spring Cloud Netflix
## Eureka
Spring Cloud Netflix的一部分，用于服务注册与发现。

Eureka acts as a central registry for all services in a microservice application. Eureka itself can be thought of as a microservice whose purpose is to help the other services discover each other.

Eureka是如何实现服务注册和发现的呢？Eureka exposes a REST API, through which services register themselves and discover other services.

Eureka如何保证高可用性？通过多个Eureka servers，Eureka's default behavior is to attempt to make friends with other Eureka servers. It will try to fetch the service registry from other Eureka servers and even register itself as a service with other Eureka servers.

什么是self-preservtion mode? Eureka expects service instances to register themselves and to continue to send registration renewal requests every 30 seconds. Normally, if Eureka doesn't receive a renewal from a service for three renewal periods, it deregisters that instance. 如果你开启self-preservation mode，那么就不会deregisters the instance，因为没有收到renewal很可能只是网络异常，而不是服务本身down掉了。一般的，我们在开发环境关闭self-preservation mode，在生产环境开启self-preservation mode.

### Eureka基本工作流程
1. 注册服务：when a *some-service* instance starts, it will register itself by name with Eureka. There may be multiple equivalent instances of *some-service*, but all of them register with Eureka with the same name
2. 发现服务：rather than hard coding *other-service* with specific host and port information for *some-service*, *other-service* looks up *some-service* from Eureka by its name. Eureka replies with information for all instances of *some-service* that it knows about. Now *other-service* needs to make a decision, which instance of *some-service* will it use? It is best to apply some client-side, load-balancing algorithm to spread the requests around, that's where another Netflix project, Ribbon, comes into play
3. 消费服务：once Ribbon has made its choice, all that's left is for *other-service* to make requests to the instance that Ribbon chooses

### 如何使用Eureka
1. startup Eureka server
2. configure Eureka server
```properties
eureka.instance.hostname=xxx
eureka.client.fetch-registry=false
eureka.client.register-with-eureka=false
eureka.client.service-url=xxx
eureka.server.enable-self-preservation
```
3. startup Eureka client
添加spring-cloud-starter-netflix-eureka-client依赖，the Eureka client starter dependency adds all you need for discovering services via Eureka, including Eureka's client-side library, as well as the Ribbon load balancer. 
添加了这个依赖后when the application starts, it attempts to contact a Eureka server running locally and listening on port 8761 to register itself under the name *UNKNOWN*.
4. configure Eureka client
```properties
spring.application.name # 在Eureka中注册的服务名称
server.port # 一般设置为0，因为我们通过Eureka发现服务，无所谓端口号，设置为0可以避免冲突
eureka.client.service-url #与configure Eureka server中的设置类似
```
5. consuming services
Spring Cloud's Eureka client support, along with the Ribbon client-side load balancer, makes it simple to look up, select, and consume a service instance. 
一般的，我们有两种方式consume a service from Eureka: 
(1) a load-balanced **RestTemplate** 
只要你添加了spring-cloud-starter-netflix-eureka-client依赖，就可以在创建和注入RestTemplate的地方增加@LoadBalanced注解，这样你就得到了load-balanced **RestTemplate**. 有了load-balanced **RestTemplate**，你就可以不再写死URL，而是使用service name. **RestTempalte** asks Ribbon to look up a service by the service name and to select an instance. Ribbon rewrites the URL to include the host and port information for the choosen service instance and then lets **RestTemplate** proceed as usual.
(2) Feign-generated client interfaces
Feign is a REST client library that applies a unique, interface-driven approach to defining REST clients. Put simply, if you enjoy how Spring Data automatically implements repository interfaces, then you're going to love Feign.
使用流程：添加spring-cloud-starter-openfeign依赖；add the @EnableFeignClients annotation to one of the configuration classes；定义interfaces, Feign automatically creates an implementation and exposes it as a bean in the Spring application context

## Ribbon
Ribbon is a client-side load balancer that makes the choice on behalf of some service. 

Often, load balancers are thought of as a single centralized service that handles all requests and distributes them across many instances of the intended target. In constrast, Ribbon is a client-side load balancer that's local to each client making the requests.

As a client-side load balancer, Ribbon has several benefits over a centralized load balancer. Because there's one load balancer local to each client, the load balancer naturally scales proportional to the number of clients. Furthermore, each load balancer can be configured to employ a load-balancing algorithm best suited for each client, rather than apply the same configuration for all services.

## Hystrix
The code we write will fail, and what is important is that when it fails, it fails gracefully. This is even more significant in the context of microservices, where it is important to avoid letting failures cascade across a distributed call stack.

Latency is also an important concern in microservices, and it is crucial that an excessively slow method not drag down the performance of the microservice, resulting in cascading latency to upstream services.

The circuit breaker pattern is an incredibly powerful means of gracefully handling failures and latency in code.

A software circuit breaker starts in a closed state, allowing invocations of a method. If, for any reason, that method fails, the circuit opens and invocations are no longer performed against the failing method. It also provides fallback behavior and is self-correcting.

Netflix Hystrix is a Java implementation of the circuit breaker pattern. Put simply, a Hystrix circuit breaker is implemented as an aspect applied to a method that triggers a fallback method should the target method fail. And, to properly implement the circuit breaker pattern, the aspect also tracks how frequently the target method fails and then forwards all requests to the fallback if the failure rate exceeds some threshold.

Deciding where to declare circuit breakers in your code is a matter of identifying methods that are subject to failure.
- methods that make REST calls
- methods that perform database queries
- methods that are potentially slow

## 如何使用Hystrix?
Before you can declare circuit breakers, you'll need to add the Spring Cloud Netflix Hystrix starter to the build specification of each of the services.
```properties
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

With the Hystrix starter dependency in place, the next thing you'll need to do is to enable Hystrix.
```java
@EnableHystrix
@SpringBootApplication
public class IngredientServiceApplication {
	public static void main(String[] args) {
		SpringApplication.run(IngredientServiceApplication.class, args);
	}
}
```

Then the @HystrixCommand comes into play. Any method that's annotated with @HystrixCommand will be declared as having a circuit breaker aspect applied to it.
```java
@HystrixCommand(fallbackMethod = "defaultIngredients",
                commandProperties = {
                  @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "500"),
            			@HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "30"),
            			@HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "25"),
            			@HystrixProperty(name = "metrics.rollingStats.timeInMilliseconds", value = "20000"),
            			@HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "60000")
                })
@GetMapping("/all")
public ResponseEntity<Map<String, List<Ingredient>>> allIngredients() {
    ...
}
```

### monitoring Hystrix
Every time a circuit breaker protected method is invoked, several pieces of data are collected about the invocation and published in an HTTP stream that can be used to monitor the health of the running application in real time.

The Hystrix stream is provided by an Actuator endpoint, it is exposed at the path */actuator/hystrix.stream*.

Application startup exposes the Hystrix stream, which can then be consumed using any REST client you want. Although writing your own Hystrix stream presentation client isn't an impossible task, perhaps you should consider using the Hystrix dashboard before expending much effort on your own dashboard.

# Spring Cloud Config Server
前面我们讲过Spring有多种渠道可以用来配置参数：
- if a configuration property is likely to change or be unique to the runtime environment, Java system properties or operating system environment variables are a fitting choice. However, you must accept that changing those properties will require the application to be restarted
- for properties that are unlikely to change and are specific to a given application, placing those property specifications in application.yml or application.properties to be deployed with the packaged application is a fine choice. However, you must completely rebuild and redeploy the application should those properties need to change

对于微服务，property management is spread across multiple codebases and deployment instances, making it unreasonable to apply the same change in every single instance of multiple services in a running application. 另外，对于密码密钥等敏感数据，配置起来就更麻烦，即使加解密也不一定满足安全需要。

所以，我们需要centralized configuration management:
- configuration is no longer packaged and deployed with the application code, making it possible to change or roll back configuration without rebuilding or redeploying the application
- microservices that share common configuration needn't manage their own copy of the property settings and can share the same properties
- sensitive configuration details can be encrypted and maintained separate from the application code. The unencrypted values can be made available to the application on demand, rather than requiring the application to carry code that decrypts the information

Spring Clound Config Server provides centralized configuration with a server that all microservices within an application can rely on for their configuration. It is not only a one-stop shop for configuration that's common across all services, but it's also able to serve configuration that's specific to a given service.

Config Server can be considered just another microservice whose role is to serve configuration data for other services in the application.

Config Server exposes a REST API through which clients can consume configuration properties.

## 如何使用Config Server
1. start up Config Server

2. configure Config Server
```properties
server.port

spring.cloud.config.server.git.uri
spring.cloud.config.server.git.search-paths
spring.cloud.config.server.git.default-label
spring.cloud.config.server.git.username
spring.cloud.config.server.git.password
spring.cloud.config.server.encrypt.enabled
```

3. setup Config Server client
添加spring-cloud-starter-config依赖及相关配置：
```properties
spring.application.name
spring.profiles.active
spring.cloud.config.uri
```
Now that you have a centralized configuration server, almost all configuration will be served from there, and each microservice won't need to carry much of its own configuration. Typically, you will only need to set *spring.application.name* to identify the application to the configuration server and *spring.cloud.config.uri* to specify the location of the configuration server. 

When the application starts up, the property source provided by the Config Server client will make a request to the Config Server. Whatever properties it receives will be made available in the application's environment.

```http
http://localhost:8888/application/default/master
```
- application: 与spring.application.name的值一致，用来提供application-specific的配置
- default: the active Spring profile，用来提供profile-specific的配置
- master: git label or branch, 可选项，默认值master

No matter what an application is named, all applications will receive configuration properties from the *application.yml* file. But each service application's *spring.application.name* property will be sent in the request to Config Server, and if there's a matching configuration file, those properties will also be returned. In the event of duplicate property definitions between the common properties in application.yml and those in an application-specific configuration file, the application-specific properties will take precedence.

## refreshing configuration properties on the fly
Traditionally, application maintenance, including configuration changes, has required that an application be redeployed or at least restarted.

Spring Cloud Config Server supports the ability to refresh configuration properties of running applications with zero downtime. Once the changes have been pushed to the backing Git repository or Vault secret store, each microservice in the application can immediately be refreshed with the new configuration in one of two ways:
- mannul: the Config Server client enables a special Actuator endpoint at /actuator/refresh. An HTTP POST request to that endpoint on each service will force the config client to retrieve the latest configuration from its backends
- automatice: a commit hook in the Git repository can trigger a refresh on all services that are clients of the Config Server. This involves another Spring Cloud project called Spring Cloud Bus for communicating between the Config Server and its clients

### manually refreshing configuration properties
```properties
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-config</artifactId>
</dependency>
		
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
Whenever you enable an application to be a client of the Config Server, the autoconfiguration in play also configures a special Actuator endpoint for refreshing configuration properties.

With the Actuator in play in the running config client application, you can refresh the configuration properties from the backend repositories any time you wish by submitting an HTTP POST request to /actuator/refresh.

### automatically refreshing configuration properties
Config Server can automatically notify all clients of changes to configuration by way of another Spring Cloud project called Spring Cloud Bus.

1. creating a webhook
A webhook is created on the configuration Git repository to notify Config Server of any changes to the Git repository.

2. handling webhook updates in Config Server
```properties
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-config-monitor</artifactId>
		</dependency>
		
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-stream-kafka</artifactId
		</dependency>
```
Enabling the /monitor endpoint in Config Server is a simple matter of adding the spring-cloud-config-monitor dependency to the Config Server's build.

The /monitor endpoint uses Spring Cloud Stream to publish notification messages to participating Config Server clients. To avoid being hardcoded to any particular messaging implementation, the monitor acts as a Spring Cloud Stream source, publishing messages into the stream and letting the underlying binding mechanism deal with the specifics of sending the messages.

Each Git implementation has its own take on what the webhook **POST** request should look like. This makes it important for the /monitor endpoint to be able to understand different data formats when handling webhook **POST** requests. Out of the box, Config Server comes with support for several popular Git implementations, such as GitHub and GitLab. If you are using one of these Git implementations, nothing special is required. Otherwise, you need to provide a specific notification extractor for your Git implementation.

3. enabling auto-refresh in Config Server clients
```properties
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-bus-kafka</artifactId>
</dependency>
```
With the appropriate Spring Cloud Bus starter in place, autoconfiguration will kick in as the application starts up and will automatically bind itself to a RabbitMQ broker or Kafka cluster running locally. 

Config Server client applications subscribed to the notifications react to the notification messages by refreshing their environments with new property values from the Config Server.

# Spring Cloud Stream

