If a transaction rolls back, we need to ensure [[W6N3 - Transactions#Transaction guarantees (ACID)|atomicity]] and make sure the DB is restored to the state it was in before the transaction occurred. If the entire DBMS crashes, when it restarts it must make sure that all committed transactions are durable, while any in-progress transactions are correctly aborted.
# Types of failures
There are three categories of failures:
- **Transaction failures**: a number of transactions fail but the system remains operational
	- **Logical errors**: a transaction cannot complete due to an internal error condition, e.g. an integrity constraint violation
	- **Internal state errors**: the DBMS terminates an active transaction due to an error condition, e.g. a [[W7N1 - Locking#Deadlocks|deadlock]]
- **System failures**: something goes wrong and the entire DBMS crashes
	- **Software failures**: a bug in the DBMS implementation causes it to crash
	- **Hardware failure**: the computer hosting the DBMS crashes, e.g. due to a power cut
- **Non-repairable hardware failures**: e.g. a disk failure destroys part or all of non-volatile storage. This is assumed to be detectable, either from system non-response or from checksums detecting data corruption, but is not recoverable by the DBMS and requires the database to be restored from a backup or replica.
# Managing the buffer pool
The DBMS needs to guarantee that the changes of a transaction are durable once the DBMS has confirmed that it committed, and that no partial changes are durable if the transaction aborts. The way this is supported depends on how it manages the [[W2N4 - Buffer management|buffer pool]], specifically two policies:
- **Steal policy**: whether the DBMS allows frames with uncommitted updates to be replaced and therefore flushed to storage.
- **Force policy**: whether the DBMS requires that all updates made by a transaction are reflected on non-volatile storage before the transaction is allowed to commit.
## No-steal and force
This approach is easiest to implement, as we never have to undo the changes of an aborted transaction as they are not written to disk, and we never have to redo changes of a committed transaction because they are guaranteed to be written to disk on commit. It has significant drawbacks, as flushing non-contiguous pages is slow, and the DBMS may still crash part way through flushing, and requires that all modified pages can fit in the buffer pool.
## Steal and no-force
**Steal and no-force** is used by most DBMSes, as it offers the best runtime performance at the cost of slow recovery (but as recovery is a relatively uncommon event this is an acceptable cost). This requires remembering the previous value of changes made when a steal occurs, and storing data to allow for redoing modifications on commit using a write ahead log.
### Write ahead logging
Before any change is made to the database, the change is recorded in a **write-ahead log (WAL)** file. The log records the action, the target, and the previous and new values of that target. This allows for redoing an operation with only the information in the log, and to roll back the operation. A transaction is not considered committed until all of its log records including its commit record are written to non-volatile storage.

E.g.
For a schedule `T1`:
```
BEGIN
W(A)
W(B)
COMMIT
```
the WAL will look like:
```
<T1, BEGIN>
<T1, A, 1, 8> // A was 1, and was changed to 8
<T1, B, 5, 9> // B was 5, and was changed to 9
<T1, ABORT>
```
This is enough to either redo the transaction, or to revert it if needed.
# ARIES
ARIES is a recovery algorithm developed by IBM Research in the 90s. It uses steal and no-force, with a write-ahead log, and performs recovery in 3 phases:
1. **Analyse**: identify active transactions and dirty pages at the time of crash
2. **Redo**: repeat history to restore the exact state just before the crash
3. **Undo**: reverse the actions of all transactions that did not commit before the crash