# Concurrency Control in Distributed Transactions

Concurrency control mechanisms provide us with various concepts & implementations to ensure the execution of any transaction across any node doesn’t violate ACID or BASE (depending on database) properties causing inconsistency & mixup of data in the distributed systems. Transactions in the distributed system are executed in “sets“, every set consists of various sub-transactions. These sub-transactions across every node must be executed serially to maintain data integrity & the concurrency control mechanisms do this serial execution.

### Types of Concurrency Control Mechanisms

#### Pessimistic Concurrency Control (PCC):
Pessimistic Concurrency Control (PCC) is a type of concurrency control mechanism that assumes conflicts between transactions are likely to occur and thus prevents these conflicts by locking resources before they are accessed. PCC ensures data integrity and consistency by preventing other transactions from accessing data that is currently being processed.

#### . . . . Operations

1. **Lock Acquisition:**

- When a transaction needs to read or write a data item, it must first acquire the appropriate lock.

- If the lock is already held by another transaction, the requesting transaction is blocked until the lock is released.

2. **Conflict Resolution:**

- PCC handles conflicts by blocking transactions that request locks on data items already locked by other transactions.

- Deadlocks can occur if transactions wait indefinitely for locks. Deadlock detection and resolution mechanisms, such as timeout or wait-for graph analysis, are used to handle this.

3. **Lock Release:**

- Locks are typically released when the transaction commits or aborts.
- In strict 2PL, all locks are held until the transaction commits, ensuring that no other transaction can see intermediate states.

### Pessimistic Concurrency Control Methods

Following are the four Pessimistic Concurrency Control Methods:

#### 1. Operation of Two-Phase Locking

1. **Growing Phase:**

- During the growing phase, a transaction requests the locks it needs. If a requested lock is available, it is granted. If not, the transaction waits until the lock can be obtained.

- The transaction continues to acquire locks until it reaches the point where it no longer needs any additional locks.

2. **Lock Point:**

- The point in the execution of a transaction at which it has acquired all the locks it will ever need. After this point, the transaction will not request any more locks.

3. **Shrinking Phase:**

- After reaching the lock point, the transaction begins to release its locks as it finishes its operations on the data items.

- Once a lock is released, the transaction cannot acquire any more locks.

#### 2. ACID ISOLATION LEVEL

#### 3. Distributed Lock Manager (DLM):

- The DLM is responsible for managing locks across a distributed system. It ensures that locks are granted and released in a way that prevents conflicts and ensures data consistency.
- The DLM can be centralized (a single node managing all locks) or decentralized (locks managed by multiple nodes, often in a coordinated manner).

#### 4. Multiple Granularity Lock

A lock can be acquired at various granular level like: table level, row/record level, page level or any other resource’s level. In transaction system a transaction can lock a whole table, or a specific row while performing some changes on it. This lock acquiring when done by various transactions simultaneously, this phenomena is called as multiple granularity locking.

#### Optimistic Concurrency Control (OCC)

Optimistic Concurrency Control (OCC) is a concurrency control method used in database systems that assumes conflicts between transactions are rare. Instead of using locks to prevent conflicts, OCC allows transactions to execute without restrictions and only checks for conflicts at the time of transaction commit. If a conflict is detected, one or more transactions are rolled back and retried. This approach can lead to higher concurrency and better performance in environments where conflicts are infrequent.

#### . . . . Operation of Optimistic Concurrency Control

1. **Read Phase:**
- The transaction reads data items from the database.
- All read and write operations are performed on a private workspace or local copies of the data items.
- No locks are acquired on the data items, which allows multiple transactions to read and perform operations concurrently without waiting for locks.

2. **Validation Phase:**

##### Actions:
- Before committing, the transaction checks if there have been any conflicts with other transactions.
- The transaction maintains three sets of data items:
  - Read Set (RS): The set of data items read by the transaction.
  - Write Set (WS): The set of data items written by the transaction.
  - Validation Set (VS): The set of data items validated by the transaction.

##### Validation Process:

- **Timestamp Assignment:** Each transaction is assigned a unique timestamp when it enters the validation phase.
- **Conflict Check:** The transaction checks for conflicts with other transactions that have committed since it started. This typically involves:
  - Ensuring that no other transactions have written to any data items in its read set (RS) after the transaction read them.
  - Ensuring that no other transactions have read any data items in its write set (WS) before it commits.


3. **Write Phase:**

- If validation is successful (no conflicts detected), the transaction proceeds to commit its changes.
- The changes from the private workspace or local copies are written to the database.
- If validation fails (conflict detected), the transaction is rolled back, and its changes are discarded. The transaction may be retried.


### Optimistic Concurrency Control Methods

#### 1. Timestamp Based (OCC)

In a timestamp based concurrency technique, each transaction in the system is assigned a unique timestamp which is taken as soon as the transaction begins, and its verified again during the commit phase. If there’s new updated timestamp from a different transaction then based on some policy defined by the System Adminstrator the transaction will either be restarted or aborted. But if the times stamp is same & never modified by any other transaction then it will be committed.

#### 2. Multi-Version Concurrency Control (MVCC)

In MVCC, every data item has multiple versions of itself. When a transaction starts, it reads the version that is valid at the start of the transaction. And when the transaction writes, it creates a new version of that specific data item. That way, every transaction can concurrently perform their operations.

#### 3. Snapshot Isolation
Snapshot isolation is basically a snapshot stored in an isolated manner when our database system was purely consistent. And this snapshot is read by the transactions at the beginning. Transaction ensures that the data item is not changed while it was executing operations on it. Snapshot isolation is achieved through OCC & MVCC techniques.

#### 4. Conflict Free Replicated Data Types (CRDTs)

CRDTs is a data structure technique which allows a transaction to perform all its operation and replicate the data to some other node or current node. After all the operations are performed, this technique offers us with merging methods that allows us to merge the data across distributed nodes (conflict-free) and eventually achieving consistent state (eventually consistent property).

## Two-Phase Commit (2PC)

The Two-Phase Commit (2PC) protocol is a distributed algorithm used to ensure all nodes in a distributed system agree on the outcome of a transaction. It is designed to maintain the ACID properties of transactions across multiple databases or services. The protocol consists of two phases: the prepare phase and the commit phase.

#### . . . . Key Components

1. **Coordinator:** the node or process that manages the transaction and coordinates the commit process across all participating nodes (participants).
2. **Participants:** The nodes or processes that execute parts of the transaction and participate in the commit process.

### Phases of 2PC

#### 1. Prepare Phase (Phase 1):
- ##### Coordinator Actions:
  1. The coordinator sends a "prepare" message to all participants.
  2. The message requests that each participant votes on whether it can commit the transaction or not.

- #### Participant Actions:
   1. Each participant receives the "prepare" message.
   2. Each participant performs all necessary actions to ensure it can commit the transaction (e.g., locking resources, performing preliminary checks).
   3. Each participant responds to the coordinator with either a "vote-commit" if it can commit or a "vote-abort" if it cannot.

#### Commit Phase (Phase 2):
- ##### Coordinator Actions:
   1. The coordinator collects votes from all participants.
   2. If all participants vote to commit, the coordinator sends a "commit" message to all participants.
   3. If any participant votes to abort, the coordinator sends an "abort" message to all participants.
- ##### Participant Actions:
   1. Each participant receives the "commit" or "abort" message from the coordinator.
   2. If the message is "commit," each participant commits the transaction.
   3. If the message is "abort," each participant aborts the transaction and rolls back any changes made during the prepare phase.


#### . . . . Detailed Example of 2PC

#### Scenario:
A transaction involves two databases, DB1 and DB2, coordinated by a transaction manager (coordinator).

##### 1. Prepare Phase:

  - Coordinator: Sends "prepare" message to DB1 and DB2.
  - **DB1:**
    - Receives "prepare" message.
    - Checks if it can commit the transaction (e.g., locks necessary data, ensures no conflicts).
    - Sends "vote-commit" to the coordinator if it can commit, otherwise sends "vote-abort."
  - **DB2:**
    - Receives "prepare" message.
    - Checks if it can commit the transaction.
    - Sends "vote-commit" to the coordinator if it can commit, otherwise sends "vote-abort."

##### 2. Commit Phase:
  - **Coordinator:** 
   - Receives votes from DB1 and DB2.
   - If both votes are "commit," sends "commit" message to DB1 and DB2.
   - If any vote is "abort," sends "abort" message to DB1 and DB2.

  - **DB1 and DB2:**
    - Receive the "commit" or "abort" message.
    - If "commit," both DB1 and DB2 commit the transaction.
    - If "abort," both DB1 and DB2 roll back the transaction.

#### . . . .Handling Failures
2PC must handle various failure scenarios to ensure consistency:

#### 1. Participant Failure During Prepare:
If a participant fails before responding to the prepare message, the coordinator can timeout and decide to abort the transaction, sending an "abort" message to all participants.

#### 2.Coordinator Failure:
If the coordinator fails after sending the prepare messages but before sending the commit/abort messages, participants may be left in an uncertain state. Participants must wait for the coordinator to recover and resend the decision.

#### 3. Participant Failure During Commit:
If a participant fails after sending a "vote-commit" but before receiving the final commit/abort decision, it must consult the coordinator upon recovery to determine the transaction's outcome.