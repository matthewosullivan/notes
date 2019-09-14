# notes

## Patterns for distributed transactions within a microservices architecture

One common problem is how to manage distributed transactions across multiple microservices.

## What is a distributed transaction?

When a microservice architecture decomposes a monolithic system into self-encapsulated services, it can break transactions. This means a local transaction in the monolithic system is now distributed into multiple services that will be called in a sequence.

In a monolithic system, the system will create a local database transaction that works over multiple database tables. If any step fails, the transaction can roll back. This is know as ACID (Atomicity, Consistency, Isolation, Durability), which is guaranteed by the database system.

When we decompose the system, each service has a separate database.

Because the transaction is now across multiple databases, it is now considered a distributed transaction.

## What is the problem?

In a database system, atomicity means that in a transaction either all steps complete or no steps complete. The microservice-based system does not have a global transaction coordinator by default. in the example above, if the CreateOrder method fails, how do we roll back changes applied by other services?

## Do we isolate user actions for concurrent requests?

If an object is written by a transaction and at the same time (before the transaction ends), it is read by another request, should the object return old data or updated data?

## Possible solutions

Two patterns can resolve the problem:

- 2pc (two-phase commit)
- Saga

## Two-phase commit (2pc) pattern

2pc is widely used in database systems. For some situations, you can use 2pc for microservices for microservices. 

What is a two-phase commit?

2pc has two phases: A prepare phase and a commit phase. In the prepare phase, all microservices will be asked to prepare for some data change that could be done atomically. Once all microservices are prepared the commit phase will ask all the microservices to make the actual changes.

Normally, there needs to be a global coordinator to maintain the lifecycle of the transaction, and the coordinator will need to call the microservices in the prepare and commit phases.

The coordinator will first create a global transaction with all the context information. It will then tell the microservices to prepare with the created transaction. Microservices will lock down objects from further changes and tell the coordinator that it is prepared. Once the coordinator has confirmed all microservices are ready to apply their changes, it will then ask them to apply their changes by requesting a commit with the transaction. At this point all objects will be unlocked.

If at any point a single microservice fails to prepare, the Coordinator will abort the transaction and begin the rollback process. The coordinator will request an abort on the microservice, and the microservice will then roll back any changes made, and unlock the database objects.

Benefits of using 2pc

The prepare and commit phases guaranteee that the transaction is atomic..

2pc allows read-write isolation. This means the changes on a field are not visible until the coordinator commits the changes.

Disadvantages of using 2pc

not recommended for many microservice-based systems because 2pc is synchronous (blocking). The protocol will need to lock the object that will be changed before the transaction completes. 

In a database system, transactions tend to be fast-normally within 50 ms. However, microservices have long delays with RPC calls, especially when integrating with external services such as payment service. The lock could become a system bottleneck. Also, it is possible to have two transactions mutally lock each other (deadlock) when each transaction requests a lock on a resource the other resource requires.

Saga pattern

The Saga pattern is another widely used pattern for distributed transactions.
The Saga pattern is asynchronous and reactive. In a Saga pattern, the distributed transaction is fulfilled by asynchronous local transactions on all related microservices. The microservices communicate with each other through an event bus.

Saga pattern for the customer order example

- Order Microservice - createOrder - emits an OrderCreated event
- Customer Microservice - listens for OrderCreated event and updates a customer fund once the event is received. If a deduction is successfully made from a fund, a CustomerFundUpdated event will then be emitted, which in this examples means the end of the transaction.

If any microservice fails to complete its local transactin, the other microservice will run compensation transactions to rollback changes.

Saga pattern for a compensation transaction:

- If the UpdateCustomerFund failed for some reason and it then emitted a CustomerFundUpdateFailed event. The OrderMicroservice listens for the event and start its compensation transaction to revert the order that was created.

## Advantages of the Saga pattern

- Support for long-lived transactions

## Disadvantages of the Saga pattern

- Does not have read isolation, customer could see the order being created, but in the next second, the order is removed due to a compensation transaction.

Adding a process manager

To address issues of the Saga pattern, it is quite normal to add a process manager as an orchestrator. The process manager is responsible for listening to events and triggering endpoints.

Conclusion

The Saga pattern is a preferrable way of solving distributed transaction problems for microservice-based architecture. 

It introduces a new set of problems:
- How to atomically update the database and emit an event






