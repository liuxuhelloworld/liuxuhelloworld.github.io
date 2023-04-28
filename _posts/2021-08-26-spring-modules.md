Spring has many modules, but you need only a subset of these modules depending on your application's needs. Each module has its compiled binary code in a JAR file along with corresponding Javadoc and source JARs.

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
- message: this module contains key abstractions taken from the Spring Integration project to serve as a foundation for message-based applications and adds support for STOMP messages
- orm: this module extends Spring's standard JDBC feature set with support for popular ORM tools including Hibernate, JDO, JPA, and the data mapper iBATIS
- test: Spring provides a set of mock classes to aid in testing your applications, and many of these mock classes are used within the Spring test suite, so they are well tested and make testing your applications much simpler
- tx: this module provides all classes for supporting Spring's transaction infrastructure
- web: this module contains the core classes for using Spring in your web applications
- webmvc: this module contains all the classes for Spring's own MVC framework
- websocket: this module provides support for JSR-356 (Java API for WebSocket)