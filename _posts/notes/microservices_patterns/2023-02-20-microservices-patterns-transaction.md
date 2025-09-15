The traditional approach to maintaining data consistency across multiple services, databases, or message brokers is to use distributed transactions. The de facto standard for distributed transaction management is the X/Open Distributed Transaction Processing (DTP) Model, X/Open XA. XA uses *two-phase commit* (2PC) to ensure that all participants in a transaction either commit or rollback. An XA-compliant technology stack consists of XA-compliant databases and message brokers, database drivers, and messaging APIs, and an interprocess communication mechanism that propagates the XA global transaction ID. Most SQL databases are XA compliant, as are some message brokers.

As simple as this sounds, there are a variety of problems with distributed transactions. One problem is that many modern technologies, including NoSQL databases such as MongoDB and Cassandra, don't support them. Also, distributed transactions aren't supported by modern message brokers such as RabbitMQ and Apache Kafka. Another problem with distributed transactions is that they are a form of synchronous IPC, which reduces availability. In order for a distributed transaction to commit, all the participanting services must be available. There is even Eric Brewer's CAP theorem, which states that a system can only have two of the following three properties: consistency, availability, and partition tolerance. Today, architects prefer to have a system that's available rather than one that's consistent.

To solve the more complex problem of maintaining data consistency in a microservice architecture, an application must use a different mechanism that builds on the concept of loosely coupled, asynchronous services. This is where sagas come in.

# Saga
Instead of an ACID transactions, an operation that spans services must use what's known as a *saga*, a message-driven sequence of local transactions, to maintain data consistency. 

Saga: maintain data consistency across services using a sequence of local transactions that are coordinated using asynchronous messaging.

![Saga example](/assets/images/microservices_patterns/microservices-patterns-transaction-saga-example.jpeg)

A saga consists of three types of transactions:
- compensatable transactions, transactions that can potentially be rolled back using a compensating transaction
- pivot transaction, the go/no-go point in a saga. If the pivot transaction commits, the saga will run until completion. A pivot transaction can be a transaction that's neither compensatable nor retriable. Alternatively, it can be the last compensatable transaction or the first retriable transaction
- retriable transactions, transactions that follow the pivot transaction and are guaranteed to succeed

![saga transaction types](/assets/images/microservices_patterns/microservices-patterns-transaction-saga-transaction-types.jpeg)

Sagas differ from ACID transactions in a couple of important ways. They lack the isolation property of ACID transactions. Also, because each local transaction commits its changes, a sage must be rolled back using compensating transactions.

# Coordinating Sagas
There are mainly two ways to structure a saga's coordination logic:
- choreography: distribute the decision making and sequencing among the saga participants. They primarily communicate by exchanging events
- orchestration: centralize a saga's coordination logic in a saga orchestrator class. A saga *orchestrator* sends command messages to saga participants telling them which operations to perform

## choreography-based sagas
When using choreography, there's no central coordinator telling the saga participants what to do. Instead, the saga participants subscribe to each other's events and respond accordingly.

![choreogrphy based saga](/assets/images/microservices_patterns/microservices-patterns-transaction-saga-choreography.jpeg)

Choreography can work well for simpla sagas, it is often better for more complex sagas to use orchestration.

## orchestration-based sagas
When using orchestration, you define an orchestrator class whose sole responsibility is to tell the saga participants what to do. The saga orchestrator communicates with the participants using command/async-reply style interaction. To execute a saga step, it sends a command message to a participant telling it what operation to perform. After the saga participant has performed the operation, it sends a reply message to the orchestrator. The orchestrator then processes the message and determines which saga step to perform next.

![orchestration based saga](/assets/images/microservices_patterns/microservices-patterns-transaction-saga-orchestration.jpeg)

A good way to model a saga orchestrator is as a state machine. A *state machine* consists of a set of states and a set of transitions between states that are triggered by events. Each transition can have an action, which for a saga is the invocation of a saga participant. The transitions between states are triggered by the completion of a local transaction performed by a saga participant. The current state and the specific outcome of the local transaction determine the state transition and what action, if any, to perform. There are also effective testing strategies for state machines. As a result, using a state machine model makes designing, implementing, and testing sagas easier.

One benefit of orchestration is that it doesn't introduce cyclic dependencies. The saga orchestrator invokes the saga participants, but the participants don't invoke the orchestrator. As a result, the orchestrator depends on the participants but not vice versa, and so there are no cyclic dependencies.

Orchestration has a drawback: the risk of centralizing too much business logic in the orchestrator. This results in a design where the smart orchestrator tells the dumb services what operations to do. Fortunately, you can avoid this problem by designing orchestrators that are solely responsible for sequencing and don't contain any other business logic.

# Lack of Isolation
Saga's ACD:
- atomicity: the saga implementation ensures that all transactions are executed or all changes are undone
- consistency: referential integrity within a service is handled by local databases. Referential integrity across services is handled by services
- durability: handled by local databases

The isolation property of ACID transactions ensures that the outcome of executing multiple transactions concurrently is the same as if they were executed some serial order. The database provides the illusion that each ACID transaction has exclusive access to the data. Isolation makes it a lot easier to write business logic that executes concurrently.

This lack of isolation potentially causes what the database literature calls *anomalies*. An anomaly is when a transaction reads or writes data in a way that it wouldn't if transactions were executed one at time. When an anomaly occurs, the outcome of executing sagas concurrently is different than if they were executed serially.

On the surface, the lack of isolation sounds unworkable. But in practice, it's common for developers to accept reduced isolation in return for higher performance. An RDBMS lets you specify the isolation level for each transaction. The default isolation level is usually an isolation level that's weaker than full isolation, also known as serializable transactions. Real-world database transactions are often different from textbook definitions of ACID transactions.

The lack of isolation can cause the following three anomalies:
- lost updates, a lost update anomaly occurs when one saga overwrites an update made by another saga
- dirty reads, a dirty read occurs when one saga reads data that's in the middle of being updated by another saga
- fuzzy/nonrepeatable reads, two different steps of a saga read the same data and get different results because another saga has made updates

One challenge with sagas is that they are ACD. They lack the isolation feature of traditional ACID transactions. That's because the updates made by each of a saga's local transactions are immediately visible to other sagas once that transaction commits. As a result, an application must use what are known as *countermeasures*, design techniques that prevent or reduce the impact of concurrency anomalies caused by the lack of isolation.

## semantic lock
When using the semantic lock countermeasure, a saga's compensatable transaction sets a flag in any record that it creates or updates. The flag indicates that the record isn't *commited* and could potentially change. The flag can either be a lock that prevents other transactions from accessing the record or a warning that indicates that other transactions should treat that record with suspicion. It is cleared by either a retriable transaction (the saga is completing successfully), or by a compensating transaction (the saga is rolling back).

The **_PENDING** states, such as **APPROVAL_PENDING** and **REVISION_PENDING**, implement a semantic lock.

Managing the lock is only half the problem. You also need to decide on a case-by-case basis how a saga should deal with a record that has been locked.

A benefit of using semantic locks is that they essentially recreate the isolation provided by ACID transactions. Sagas that update the same record are serialized, which significantly reduces the programming effort. The drawback is that the application must mange locks. It must also implement a deadlock detection algorithm that performs a rollback of a saga to break a deadlock and re-execute it.

## commutative updates
One straightforward countermeasure is to design the update operations to be commutative. Operations are *commutative* if they can be executed in any order.

## pessimistic view
Another way to deal with the lack of isolation is the *pessimistic view* countermeasure. It reorders the steps of a saga to minimize business risk due to a dirty read.

## reread value
The *reread value* countermeasure prevents lost updates. A saga that uses this countermeasure rereads a value before updating it, verifies that it's unchanged, and then updates the record. If the record has changed, that saga aborts and possibly restarts.

## version file
The *version file* countermeasure is so named because it records the operations that are performed on a record so that it can reorder them. It is a way to trun noncommutative operations into commutative operations.

# Saga example
![create order saga](/assets/images/microservices_patterns/microservices-patterns-transaction-create-order-saga.jpeg)