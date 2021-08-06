Spring Boot applications tend to bring everything they need with them and don't need to be deployed to some application server.

# Spring Boot DevTools
## automatic application restart
DevTools会监测代码或配置文件的变动，并自动重启应用。准确地说，当你使用DevTools时，the application is loaded into two separate class loaders in the Java virtual machine (JVM). One class loader is loaded with your Java code, property files, and pretty much anything that is in the src/main/ path of the project. The other class loader is loaded with dependency libraries, which aren't likely to change as often. When a change is detected, DevTools reloads only the class loader containing your project code and restarts the Spring application context, but leaves the other class loader and the JVM intact.

## automatic browser refresh  and template cache disable
Template cache是默认的，这样templates don't need to be reparsed with every request they serve. DevTools帮你自动关闭template cache，这样方便调试前端内容

## built-in H2 console
http://localhost:8080/h2-console
jdbc:h2:mem:testdb

# Spring Boot Actuator
Spring Boot Actuator offers production-ready features such as monitoring and metrics to Spring Boot applications.

Actuator's features are provided by way of several endpoints, which are made available over HTTP as well as through JMX MBeans. Using endpoints exposed by Actuator, we can ask things about the internal state of a running Spring Boot application.

## 如何使用Actuator?
1. To enable Actuator in a Spring Boot application, you simply need to add Actuator's starter dependency to your build
```properties
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

2. configurations
```properties
management.endpoints.web.base-path=xxx
management.endpoints.web.exposure.include=xxx,xxx,xxx
management.endpoints.web.exposure.exclude=xxx,xxx,xxx
management.endpoint.health.show-details=always
```

3. consuming endpoints
- localhost:xxx/actuator
- localhost:xxx/actuator/info
- localhost:xxx/actuator/health
- localhost:xxx/actuator/beans
- localhost:xxx/actuator/conditions
- localhost:xxx/actuator/env
- localhost:xxx/actuator/mappings
- localhost:xxx/actuator/loggers
- localhost:xxx/actuator/httptrace
- localhost:xxx/actuator/threaddump
- localhost:xxx/actuator/heapdump
- localhost:xxx/actuator/metrics
- localhost:xxx/actuator/metrics/xxx

4. customizing Actuator
A few of the endpoints themselves allow for customization. Moreover, Actuator itself allows you to create custom endpoints.

