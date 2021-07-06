# A Short History
早期Spring通过XML文件描述components and their relationship to other components. 
```xml
<bean id="inventoryService" class="com.example.InventoryService" />
<bean id="productService" class="com.example.ProductService"
		<constructor-arg ref="inventoryService">
</bean>
```

随后Spring开始支持Java-based configuration.
```java
@Configuration
public class ServiceConfiguration {
		@Bean
		public InventoryService inventoryService() {
				return new InventoryService();
		}

		@Bean 
		public ProductService productService() { 
				return new ProductService(inventoryService()); 
		}
}
```

Spring是支持某些自动配置的，即component scanning和autowiring. With component scanning, Spring can automatically discover components from an application's classpath and create them as beans in the Spring application context. With autowireing, Spring automatically injects the components with the other beans that they depend on.

Spring Boot is an extension of the Spring Framework that offers several productivity enhancements，其中最大的enhancement就是autoconfiguration. Spring Boot can make reasonable guesses of what components need to be configured and wired together, based on entries in the classpath, environment variables, and other factors. Spring Boot autoconfiguration has dramatically reduced the amount of explicit configuration (whether with XML or Java) required to build an application.

# Property Injection vs Bean Wiring
property injection: configuration that sets values on beans in the Spring application context，即我们需要什么样的bean.

bean wiring: configuration that declares application components to be created as beans in the Spring application context and how they should be injected into each other，即如何把这些beans整合起来。

在没有autoconfiguration之前，不管是XML配置还是Java-based annotaion，property injection和bean wiring一般是一起配置的，比如下面这段代码，it declares a DataSource for an embedded H2 database and set some String properties with the name of SQL scripts that should be applied to the database once the data source is ready.

```java
		@Bean
		public DataSource dataSource() {
			return new EmbeddedDataSourceBuilder()
				.setType(H2)
				.addScript("taco_schema.sql")
				.addScripts("user_data.sql", "ingredient_data.sql")
				.build();
		}
```

有了autoconfiguration，你就不用自己定义DataSource了，if the H2 dependency is available in the run-time classpath, then Spring Boot automatically creates an appropriate DataSource bean in the Spring application context, the bean applies the SQL scripts **schema.sql** and **data.sql**.

但是，问题来了，如果除了schema.sql和data.sql，你还想执行其他的sql文件怎么办呢？所以我们需要configuration properties, configuration properties are nothing more than properties on beans in the Spring application context that can be set from one of several property sources, including JVM system properties, command-line arguments, and environment variables.

# Spring Environment Abstraction
The Spring environment abstraction is a one-stop shop for configuration properties. 也就是说，Spring environment从各个property source拿到properties，so that beans needing those properties can onsume them from Spring itself. property source包括：
- JVM system properties
- operating system environment variables
- command-line arguments
- application property configuration files

比如，你希望servlet container监听某个端口，那在可以在application.properties中定义server.port=xxx，也可以在命令行启动时指定参数--server.port=xxx，也可以设置环境变量SERVER_PORT=xxx.

In fact, there are several hundred configuration properties you can use to tweak and adjust how Spring beans behave.

# Configuration Properties
```java
@Configuration
@ConfigurationProperties(prefix = "taco.orders")
@Data
@Validated
public class TacoOrderConfig {

    @Min(value = 5, message = "must between 5 and 20")
    @Max(value = 20, message = "must between 5 and 20")
    private int pageSize = 10;
}
```
When @ConfigurationProperties is placed on any Spring bean, it specifies that the properties of that bean can be injected from properties in the Spring environment.

一般的，我们把相关的configurations归集到一起作为configuration properties holder，整洁、便于复用、方便管理。

# Spring Profile
对于开发、测试、生产环境，我们希望有不同的配置，怎么办？

一种方法是使用environment variables，不同的环境配置不同的环境变量，这种方法太简单暴力，不够灵活，也不方便维护。

一般的，我们使用Spring profile来实现不同环境的不同配置。

Spring profiles are a type of conditional configuration where different beans, configuration classes, and configuration properties are applied or ignored based on what profiles are active at runtime.

很简单，提供profile-specific的配置文件就可以了，以application-{profile name}.yml或application-{profile name}.properties的格式命名。比如，application-prod.yml、application-test.properties.

对于YAML配置文件，profile-specific的配置和non-profiled的配置可以写在一个文件里，用---分隔，并对每个profile-specific的配置声明spring.profiles即可。

如何声明当前环境使用哪一个profile-specific的配置呢？通过设置spring.profiles.active. 一般的，我们通过命令行参数或环境变量的方式设置它的值：
```bash
java -jar xxx.jar --spring.profiles.active=prod

export SPRING_PROFILES_ACTIVE=prod,audit,ha
```

profile不仅可以用来为不同环境提供不同配置，甚至可以用来控制某个bean是否被创建。
- @Profile("dev")，在dev profile是active的条件下才创建对应的bean
- @Profile({"dev", "qa"})，在dev profile或qa profile是active的条件下创建对应的bean
- @Profile("!prod")，在prod profile不是active的条件下创建对应的bean

# 配置示例（data source）
通过配置spring.datasource.xxx，你就可以配置DataSource bean.

```properties
spring.datasource.url=jdbc:mysql://localhost/tacocloud
spring.datasource.username=tacodb
spring.datasource.password=tacopassword
spring.datasource.initialization-mode=always
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.type=org.apache.tomcat.jdbc.pool.DataSource
spring.datasource.schema=classpath:xxx.sql
spring.datasource.data=classpath:xxx.sql
```

# 配置示例（embedded server）
通过配置server.xxx，你就可以配置the embedded server.

```properties
server.port=0
```
随机选择一个port，适合自动集成测试或微服务场景

```properties
server.port=8443
server.ssl.key-store: classpath: taocloud.jks
server.ssl.key-store-password:xxx
server.ssl.key-password:xxx
```
server.ssl.xxx，HTTPS相关配置项，port 8443 is the common choice for development HTTPS servers.

# 配置示例（logging）
通过配置logging.xxx，你就可以配置日志。

```properties
logging.level.xxxlogger=xxx
logging.path=xxx
logging.file=xxx
```

如果你需要full control over the logging，你可以在src/main/resources下写一个logback.xml.