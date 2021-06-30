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