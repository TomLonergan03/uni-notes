There are two approaches to guaranteeing [[W6N3 - Transactions#Conflict serialisable schedules|serialisable schedules]]:
- The optimistic approach is versioning, where a transaction is validated and possibly rolled back on commit, which is used if conflicts are rare, e.g. write-once-read-many systems.
- The pessimistic approach uses **locks** to protect database objects, and is the standard approach if conflicts are frequent.

The **lock manager** tracks which locks are granted and will make lock requests wait until the lock is released:
![[w7n1lockManager.png]]
# Basic lock types
If we lock a resource with an:
- **Shared or S-lock**: another reader can also get an S-lock on that resource, but both may only read from it.
- **Exclusive or X-lock**: we can write to it, and nobody else can get a lock.

Note that using locks alone does not guarantee a serialisable schedule:
![[w7n1lockingUnserialisable.png]]
# Two-phase locking
In order to enforce serialisable schedules, we can use **two-phase locking**.
![[w7n1twoPhaseLocking.png]]
Phase 1 is the **growing phase**, where the transaction requests the locks it needs, then phase 2 is the **shrinking phase**, where the transaction is forbidden from acquiring new locks, only releasing previously held locks. This is sufficient to guarantee conflict-serialisability.
![[w7n1twoPhaseConflictSerialisable.png]]

Two phase locking is subject to **cascading aborts**, where one transaction aborting can cause subsequent transactions to abort.

E.g. if $T_1$ aborts, all the work done by $T_2$ after getting the lock on $A$ is wasted, as otehrwise the value of $A$ is leaked from the aborted transaction, violating [[W6N3 - Transactions#Transaction guarantees (ACID)|atomicity]]:
![[w7n1cascadingAbort.png]]

In addition, 2PL may have deadlocks if two transactions are both waiting on a lock the other holds.
## Strict 2-phase locking
![[w7n1strict2pl.png]]
Strict 2PL avoids cascading aborts by only releasing locks at the end of the transaction. This means that a transaction cannot acquire a lock on a value that a concurrent transaction has modified, preventing any cascading aborts.
# Deadlocks
A deadlock occurs when there is a cycle of transactions that are each waiting for the next to release a lock before it may continue.
![[w7n1deadlock.png]]

Conservative 2PL prevents deadlocks, by requiring a transaction to wait until it can claim all locks before it begins executing, but is not used as it does not guarantee that a transaction will ever begin executing.
![[w7n1conservative2pl.png]]

There are two ways of dealing with deadlocks:
- **Deadlock detection**, where deadlocks are resolved after they occur
- **Deadlock prevention**, where deadlocks are not permitted to arise at all
## Deadlock detection
The DBMS creates a **waits-for** graph with a node for each transaction and an edge from $T_i$ to $T_j$ if $T_i$ is waiting for $T_j$ to release a lock. The system periodically checks for cycles in the graph, then if one is detected the DBMS selects a victim transaction to abort to break the cycle based on some heuristic, e.g. age, number of executed queries, number of locks currently held, number of previous restarts.

![[w7n1deadlockDetection.png]]
## Deadlock prevention
Deadlocks can be prevented by guaranteeing that for any two transactions $T_1$ and $T_2$, either $T_1$ will wait for locks held by $T_2$ or vice versa. This is done by assigning a priority to each transaction, usually older transactions are given higher priorities to prevent starvation. This prevents cycles from forming, as due to transitivity we will always have a waits-for relation that violates the priority rule in any attempted lock that would otherwise form a cycle.

There are two deadlock prevention policies:
- **Wait-die**: the requesting transaction will wait if it has a higher priority than the holding transaction, and will abort otherwise. This means that an older transaction will wait for a younger one to release its locks, while a younger transaction will abort instead of waiting for an older one to release its locks.
- **Wound-wait** (imo a better name is **kill-wait**): if the requesting transaction has a higher priority than the holding transaction, it will force the holding transaction to abort then claims the lock for itself, and otherwise it will wait. This means that an older transaction will forcibly take a lock from a younger one, while a younger one will wait for the older transaction to release its locks.

When a transaction restarts, it keeps its original priority. This prevents starvation, as in the worst case a transaction will keep aborting until it is the oldest transaction, at which point it will either wait until it gets the locks it needs (wait-die) or will kill everything preventing it from getting its locks (kill-wait)

![[w7n1deadlockPrevention.png]]
