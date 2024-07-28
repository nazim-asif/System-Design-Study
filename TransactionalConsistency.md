# Transactional Consistency: ACID, CAP and BASE

## ACID

1. **Atomicity:** This property ensures that a transaction is treated as a single, indivisible unit. Either all the operations within the transaction are completed successfully, or none of them are. If any part of the transaction fails, the entire transaction is rolled back, leaving the database in its original state. Atomicity ensures that partial transactions do not occur.
2. **Consistency:** This property ensures that a transaction brings the database from one valid state to another valid state, maintaining the predefined rules and constraints of the database. Any transaction will result in a state that adheres to all database rules, such as integrity constraints. If a transaction violates any consistency rule, it is aborted and rolled back.
3. **Isolation:** Isolation ensures that the operations of one transaction are isolated from the operations of other transactions. This means that concurrently executing transactions do not interfere with each other, maintaining data integrity and consistency even when multiple transactions are occurring simultaneously.

. . . . . . *Problems Addressed by Isolation Levels:*

i. ***Dirty Reads:***  Transaction "A" writes a record. Meanwhile, Transaction "B" reads that same record before Transaction A commits. Later, Transaction A decides to rollback and now we have changes in Transaction B that are inconsistent.

ii. ***Non-Repeatable Reads:*** Transaction "A" reads some record. Then Transaction "B" writes that same record and commits. Later Transaction A reads that same record again and may get different values because Transaction B made changes to that record and committed. This is a non-repeatable read.

iii. ***Phantom read:*** Consider two transactions, T1 and T2, operating on a table employees with a column salary. T1 starts and executes the query `SELECT * FROM employees WHERE salary > 5500;` it gives count 3 of employees.  T2 starts and inserts a new row: `INSERT INTO employees (id, name, salary) VALUES (4, 'David', 6500);`. T1 executes the same query again: `SELECT * FROM employees WHERE salary > 5500;`. now T1 gets count 4. This is Phantom read.


. . . . . *Isolation Levels:*

DEFAULT: Use the default isolation level of the underlying database. For example, postgress default isolation level is Read Committed and mysql is Repeatable Read.

READ_COMMITTED: A constant indicating that dirty reads are prevented; non-repeatable reads and phantom reads can occur.

READ_UNCOMMITTED: This isolation level states that a transaction may read data that is still uncommitted by other transactions.

REPEATABLE_READ: A constant indicating that dirty reads and non-repeatable reads are prevented; phantom reads can occur.

SERIALIZABLE: A constant indicating that dirty reads, non-repeatable reads, and phantom reads are prevented.

. . . . . *Isolation Propagation:*

REQUIRED: Indicates that the target method cannot run without an active txn. If active txn has already been started before the invocation of this method, then it will continue in the same txn or a new txn would begin soon as this method is called.

REQUIRES_NEW: Indicates that a new txn has to start every time the target method is called. If already active txn is going on, it will be suspended before starting a new one.

MANDATORY: Indicates that the target method requires an active txn to be running. If active txn is not going on, it will fail by throwing an exception.

SUPPORTS: Indicates that the target method can execute irrespective of active txn. If active txn is running, it will participate in the same txn. If executed without a txn it will still execute if no errors.
Methods which fetch data are the best candidates for this option.

NOT_SUPPORTED: Indicates that the target method doesn’t require the transaction context to be propagated.
Mostly those methods which run in a transaction but perform in-memory operations are the best candidates for this option.

NEVER: Indicates that the target method will raise an exception if executed in a transactional process.
This option is mostly not used in projects.

4. **Durability:** The Durability property guarantees that once a transaction has been committed, its effects are permanently recorded in the database, even in the event of a system failure, such as a crash or power loss.

. . . . *Durability techniques:*

○ WAL - Write ahead log

○ Asynchronous snapshot

○ AOF

***Drawback of ACID:*** The ACID properties are designed to ensure reliable transactions in a single-node database system. They provide a strong foundation for ensuring data integrity, consistency, and reliability within such systems. However, as databases became distributed across multiple nodes and geographic locations to improve performance, scalability, and fault tolerance, new challenges emerged that ACID properties alone could not address effectively. This led to the need for the CAP theorem.

# CAP

1. **Consistency:** Consistency refers to the idea that all nodes in a distributed system see the same data simultaneously. In other words, when a client reads from one node, it should receive the most recent and up-to-date value. Achieving strong consistency is desirable in systems where data integrity and correctness are critical, such as financial applications. However, ensuring strong consistency often comes at the expense of increased latency and decreased availability.
2. **Availability:** Availability implies that a distributed system should always respond to client requests, even in the presence of failures. High availability systems are designed to provide uninterrupted service, ensuring that clients can access the system and perform operations at any time. Achieving availability requires redundancy, fault tolerance mechanisms, and the ability to handle failures gracefully. However, prioritizing availability may lead to potential inconsistencies in data access across the system.
3. **Partition Tolerance:** Partition tolerance is the system's ability to continue functioning and providing availability even when network partitions occur. Network partitions happen when communication links between nodes fail, resulting in the system being split into multiple isolated sub-systems. Partition tolerance is crucial in distributed systems, as network failures are inevitable and can occur due to various reasons like network congestion, hardware failures, or software bugs.

. . . . *Key Points of CAP Theorem:*

     According to the CAP theorem, a distributed system can only guarantee two out of the three properties at any given time, but not all three. This means system designers must choose which two properties to prioritize based on the specific needs and trade-offs of their application.

**Combinations of CAP Properties**

***CP (Consistency and Partition Tolerance):***

1. Systems that choose Consistency and Partition Tolerance prioritize maintaining consistent data across all nodes, even if it means sacrificing availability during network partitions.

2. Example: HBase, MongoDB (in certain configurations).

3. Use Case: Applications requiring strong data consistency, such as banking systems, where the latest data must be accurate.

***AP (Availability and Partition Tolerance):***

1. Systems that choose Availability and Partition Tolerance prioritize maintaining availability of the system, even if it means some nodes may have outdated data.
2. Example: Cassandra, DynamoDB.
3. Use Case: Applications requiring high availability and can tolerate eventual consistency, such as social media feeds or shopping carts.

***CA (Consistency and Availability):***

1. Systems that choose Consistency and Availability can only function correctly if there are no network partitions, which is an unrealistic assumption in most distributed systems.
2. Example: Traditional relational databases (when not distributed).
3. Use Case: Applications requiring both strong consistency and high availability in a single-node setup or tightly coupled network environment.

. . . . *Why CAP Alone is Not Enough:*

The CAP theorem simplifies the complexity of distributed systems into three properties, forcing a binary choice. However, real-world systems often need a more nuanced approach:

1. ***Network Partitions Are Inevitable:*** In distributed systems, network failures will happen, so partition tolerance is essential.
2. ***Performance Needs:*** Strict consistency can slow down the system because of the need to coordinate across nodes, reducing performance and availability.
3. ***Dynamic Requirements:*** Systems often need to adapt to changing network conditions and varying levels of consistency.


# BASE

1. **Basically-Available:** A distributed system should be available to respond with some acknowledgment — even if it’s a failure message, to any incoming request.

2. **Soft-state:** The system may keep changing states as and when it receives new information.

3. **Eventually-consistent:** The components in the system may not reflect the same value/state of a record at a given point in time. They will settle it with time, eventually, though.


For example, considering the following two interconnected but independent services, the following can be a series of high-level processing steps —

1. Customer places an Order.
2. The Order service updates the details in its local database. It marks the payment status as ‘payment_initiated’.
3. The Order service sends a message to the queue for Payment service to consume.
4. The Payment service receives the message, but for some reason, the payment processing fails. It updates the details in its local database. It also sends a message to the Order service with the details.
5. The Order service updates its payment record as ‘payment_failed’, sends the alternate payment links to the user.
6. Steps 3 and 4 are carried out again when the user retries the payment. And upon successful completion of the payment, the Payment service updates its details as ‘payment_complete’ and sends another message with the updates to the Order service.
7. The Order service updates its record as ‘payment_complete’.