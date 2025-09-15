# Software Architecture
The software architecture of a computing system is the set of structures needed to reason about the system, which comprise software elements, relations among them, and properties of both.

More concretely, an application's architecture can be viewd from multiple perspectives, in the same way that a building's architecture can be viewed from structural, plumbing, electrical, and other perspectives.

Architecure is important because it enables an application to satisfy the second category of requirements: its *quality of service* requirements. There are also known as *quality attributes*. The quality of service requirements define the runtime qualities such as scalability and reliability. They also define development time qualities including maintainability, testability, and deployability. The architecture you choose for your application determines how well it meets these quality requirements.

# Architectural Style
An *architectural style* defines a family of such systems in terms of a pattern of structural organization. More specifically, an architecture style determines the vocabulary of components and connectors that can be used in instances of that style, together with a set of constraints on how they can be combined.

A particular architectural style provides a limited palette of elements (components) and relations (connectors) from which you can define a view of your application's architecture. An application typically uses a combination of architectural styles.

## layered architectural style
The classic example of an architectural style is the layered architecture. A *layered architecture* organizes software elements into layers. Each layer has a well-defined set of responsibilities.

The three-tier architecture is one of the most famous layered architecture.
- Presentation layer
- Business logic layer
- Persistence layer

## hexagonal architectural style
*Hexagonal architecture* is an alternative to the layered architectural style. The hexagonal architecture style places the business logic at the center. Instead of the presentation layer, the application has one or more *inbound adapters* that handle requests from the outside by invoking the business logic. Similarly, instead of a data persistence tier, the application has one or more *outbound adapters* that are invoked by the business logic and invoke external applications.

The business logic has one or more ports. A *port* defines a set of operations and is how the business logic interacts with what's outside of it.

Hexagonal architecture is also known as *ports and adapters architecture*.

![hexagonal architecture](/assets/images/microservices_patterns/microservices-patterns-hexagonal-architecture.jpeg)

An important benefit of the hexagonal architectural style is that it decouples the business logic from the presentation and data access logic in the adapters. The business logic doesn't depend on either the presentation logic or the data access logic. Another benefit is that it more accurately reflects the architecture of a modern application.

## Monolithic architectural style
Monolithic architecture is an architectural style that structures the application as a single executable/deployable component.

## Microservice architectural style
Microservice architecture is an architectural style that structures the application as a collection of loosely coupled, independently deployable services.

# Microservice Architecture
The high-level definition of microservice architecture is an architecture style that functionally decomposes an application into a set of services. Note that this definition doesn't say anything about size. Instead, what matters is that each service has a focused, cohesive set of responsibilities.

A *sercice* is a standalone, independently deployable software component that implements some useful functionality. Each service in a microservice architecture has its own architecture and, potentially, technology stack. But a typical service has a hexagonal architecture.

An important characteristic of the microservice architecture is that the services are loosely coupled. All interaction with a service happens via its API, which encapsulates its implementation details. This enables the implementation of the service to change without impacting its clients. Loosely coupled services are key to improving an application's development time attributes, including its maintainability and testability. They are much easier to understand, change, and test.

The requirement for services to be loosely coupled and to collaborate only via APIs prohibits services from communicating via a database. One downside of not sharing database is that maintaining data consistency and querying across services are more complex.

A key characteristic of the microservice architecture is that the services are loosely coupled and communicate only via APIs. One way to achieve loose coupling is by each service having its own datastore.

One challenge with using the microservice architecture is that there isn't a concrete, well-defined algorithm for decomposing a system into services. Another issue with using the microservice architecture is that developers must deal with the additional complexity of creating a distributed system. Implementing use cases that span multiple services requires the use of unfamiliar techniques. Each service has its own database, which makes it a challenge to implement transactions and queries that span services.

You must address numerous design and architectural issues when using the microservice architecture. What's more, many of these issues have multiple solutions, each with a different set of trade-offs.

A good way to describe the various architectural and design options and improve decision making is to use a pattern language. A *pattern* is a reusable solution to a problem that occurs in a particular context. A *pattern language* is a collection of related patterns that solve problems within a particular domain. A *software pattern* solves a software architecure or design problem by defining a set of collaborating software elements.

One reason why patterns are valuable is because a pattern must describe the context within which it applies. The idea that a solution is specific to a particular context and might not work well in other contexts is an improvement over how technology used to typically be discussed. A commonly used pattern structure includes three especially valuable sections:
- Forces: the forces of a pattern descibes the issues that you must address when solving a problem in a given context
- Resulting context: benefits, drawbacks, new issues
- Related patterns: predecessor, successor, alternative, generalization, specialization

# Microservice vs SOA
![compare-microservice-and-soa](/assets/images/microservices_patterns/microservices-patterns-compare-microservice-and-soa.jpg)