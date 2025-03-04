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