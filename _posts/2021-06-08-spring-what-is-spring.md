Typically, Spring is described as a lightweight framework for building Java applications.

# Spring container
Spring container，也称为Spring application context，是Spring的核心，它负责创建和管理application components.

Application components也称为beans，they are wired together inside the Spring application context to make a complete application.

如何将beans整合到一起呢？基于dependency injection. Rather than have components create and maintain the lifecycle of other beans that they depend on, a dependency-injected application relies on a separate entity (the container) to create and maintain all components and inject those into the beans that need them. This is done typically through constructor arguments or property accessor methods.

On top of its core container, Spring and a full portfolio of related libraries offer a web framework, a variety of data persistence options, a security framework, integration with other systems, runtime monitoring, microservice support, a reactive programming model, and many other features necessary for modern application development.