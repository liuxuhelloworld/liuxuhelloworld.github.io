Spring modules are simply JAR files that package the required code for the module. After you understand the purpose of each module, you can select the modules required in your project and include them in your code.

- aop: this module contans all the classes you need to use Spring's AOP features within your application
- aspects: this module contains all the classes for advanced integration with the AspectJ AOP library
- beans: this module contains all the classes for supporting Spring's manipulation of Spring beans
- context: this module contains classes that provide many extensions to Spring Core
- context-support: this module contains further extensions to the spring-context module
- core: this is the main module that you will need for every Spring application
- expression: this module contains all support classes for Spring Expression Language (SpEL)
- jdbc: this module includes all classes for JDBC support
- jms: this module includes all classes for JMS support
- orm: this module extends Spring's standard JDBC feature set with support for popular ORM tools including Hibernate, JDO, JPA, and the data mapper iBATIS
- tx: this module provides all classes for supporting Spring's transaction infrastructure
- web: this module contains the core classes for using Spring in your web applications
- web-mvc: this module contains all the classes for Spring's own MVC framework
- websocket: this module provides support for JSR-356 (Java API for WebSocket)