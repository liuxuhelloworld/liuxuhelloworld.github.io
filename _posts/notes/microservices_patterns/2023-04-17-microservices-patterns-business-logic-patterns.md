Developing complex business logic is always challenging. Developing complex business logic is even more challenging in a microservice architecture where the business logic is spread over multiple services.

# Procedural Transaction Script Pattern
Organize the business logic as a collection of procedural transaction scripts, one for each type of request.

![transaction script pattern example](/assets/images/microservices_patterns/microservices-patterns-business-logic-transaction-script-pattern.jpg)

In a typical transaction script-based design, one set of classes implements behavior and another set stores state. The transaction scripts are organized into classes that typically have no state. The scripts use data classes, which typically have no behavior.

This style of design is highly procedural and relies on few of the capabilities of object-oriented programming (OOP) languages. This approach works well for simple business logic. The drawback is that this tends not to be a good way to implement complex business logic.

# Domain Model Pattern
Organize the business logic as an object model consisting of classes that have state and behavior.

![domain model pattern example](/assets/images/microservices_patterns/microservices-patterns-business-logic-domain-model-pattern.jpg)

In an object-oriented design, the business logic consists of an object model, a network of relatively small classes. These classes typically correspond directly to concepts from the problem domain. In such a design some classes have only either state or behavior, but many contain both, which is the hallmark of a well-designed class.

The domain model pattern works well, but there are a number of problems with this approach, especially in a microservice architecture.

# Domain-Driven Design
DDD is a refinement of OOD and is an approach for developing complex business logic. 

DDD has some tactical patterns that are building blocks for domain models. Each pattern is a role that a class plays in a domain model and defines the characteristics of the class. The building blocks that have been widely adopted by developers include the following:
- Entity, an object that has a persistent identity. Two entities whose attributes have the same values are still different objects. In a Java EE application, classes that are persisted using JPA @Entity are usually DDD entities
- Value object, an object that is a collection of values. Two value objects whose attributes have the same values can be used interchangeably. An example of a value object is a Money class, which consists of a currency and an amount
- Factory, an object or method that implements object creation logic that's too complex to be done directly by a constructor. It can also hide the concrete classes that are instantiated. A factory might be implemented as a static method of a class
- Repository, an object that provides access to persistent entities and encapsulates the mechanism for accessing the database
- Service, an object that implements business logic that doesn't belong in an entity or a value object

![domain model pattern example](/assets/images/microservices_patterns/microservices-patterns-business-logic-domain-model-pattern-example.jpeg)

A traditional domain model is a web of interconnected classes. It doesn't explictly specify the boundaries of business objects, such as **Consumer** and **Order**.

# DDD Aggregates
Organize a domain model as a collection of aggregates, each of which is a graph of objects that can be treated as a unit.

![domain model aggregates example](/assets/images/microservices_patterns/microservices-patterns-business-logic-domain-model-aggregates.jpg)

An aggregate is a cluster of domain objects within a boundary that can be treated as a unit. It consists of a root entity and possibly one or more other entities and value objects.

Aggregates decompose a domain model into chunks, which are individually easier to understand. They also clarify the scope of operations such as load, update and delete. These operations act on the entire aggregate rather than on parts of it. An aggregate is often loaded in its entirety from the database, thereby avoiding any complications of lazy loading. Deleting an aggregate removes all of its objects from a database.

In DDD, a key part of designing a domain model is identifying aggregates, their boundaries, and their roots. The details of the aggregates' internal structure is secondary.

## aggregate rules
Rule #1: reference only the aggregate root

It requires that the root entity be the only part of an aggregate that can be referenced by classes outside of the aggregate. A client can only update an aggregate by invoking a method on the aggregate root.

Rule #2: inter-aggregate references must use primary keys

Another rule is that aggregates reference each other by identity (for example, primary key) instead of object references. The use of identity rather than object references means that the aggregates are loosely coupled. It ensures that the aggregate boundaries between aggregates are well defined and avoids accidentally updating a different aggregate. Also, if an aggregate is part of another service, there isn't a problem of object references that span services.

Rule #3: one transaction creates or updates one aggregate

Another rule that aggregates must obey is that a transaction can only create or update a single aggregate. This constraint is perfect for the microservice architecture. It ensures that a transaction is contained within a service. This constraint also matches the limited transaction model of most NoSQL databases. This rule makes it more complicated to implement operations that need to create or update multiple aggregates. But this is exactly the problem that sagas are designed to solve. Each step of the saga creates or updates exactly one aggregate.

## order service example
![order service business logic example](/assets/images/microservices_patterns/microservices-patterns-business-logic-order-service-example.jpg)

## kitchen service example
![kitchen service business logic example](/assets/images/microservices_patterns/microservices-patterns-business-logic-kitchen-service-example.jpg)

# Domain Events
An aggregate publishes a domain event when it's created or undergoes some other signigicant change.

In the context of DDD, a domain event is something that has happened to an aggregate. It's represented by a class in the domain model. An event usually represents a state change.

Event consumers may need the details when processing an event. One approach kown as *event enrichment* is for events to contain information that consumers need. It simplifies event consumers because they no longer need to request data from the service that published the event. Although event enrichment simplifies consumers, the drawback is that it risks making the event class less stable. An event class potentially needs to change whenever the requirements of its consumers change. This can reduce maintainability because this kind of change can impact multiple parts of the application. Satisfying every consumer can also be a futile effort. Fortunately, in many situations it's fairly obvious which properties to include in an event.

A service must use transactional messaging to publish events to ensure that they're published as part of the transaction that updates the aggregate in the database.