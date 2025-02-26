![[w6n3architecture.png]]

If a DBMS has multiple clients changing the same record in a table at the same time, we need a way to avoid race conditions. We also need to recover to a correct state after power failures or crashes. Both of these systems are based on transactions. 

A **transaction** is a sequence of operations (e.g. SQL queries) on a shared database to perform a higher level function which are executed as a single unit. A transaction starts with `BEGIN`, and ends with either `COMMIT` (which saves all changes) or `ABORT` (which undoes all changes as though the transaction never executed). A partial transaction must never happen.

E.g. transferring money between accounts:
```sql
BEGIN
	// check if checking balance > 100

	UPDATE Accounts
	SET balance = balance - 100
	WHERE customer_id = 1904
	AND account_type = 'Checking';

	UPDATE Accounts
	SET balance = balance + 100
	WHERE customer_id = 1904
	AND account_type = 'Savings';
COMMIT
```

We can check account balances either outside of the DBMS using another language, or inside the DBMS using stored procedures in PL/SQL or T-SQL (SQL extended with procedural constructs such as if-then-else, loops, variables, functions, etc.)
# Formal definition
A database is a fixed set of named data objects $(A,B,C,...)$, where a transaction can access object $A$ using $R(A)$ to read and $W(A)$ to write. In an RDBMS an object can be an attribute, record, page, or table.

A transaction is a sequence of read and write operations $T=\langle R(A),W(A),W(B),...\rangle$.
# Serial execution
The simplest way to perform transactions is to execute each transaction one-by-one as they arrive at the DBMS. Before a transaction starts, we must copy the entire database to a new file and make all changes to that file. If the transaction completes successfully, overwrite the old file with the new one, and if it fails simply delete the dirty copy. This is the approach used in SQLite.
# Concurrent Execution
Concurrent execution of individual transactions is preferred as it allows better resource utilisation (as one transaction can use the CPU when another is waiting for the disk) and can scale with the number of CPUs, as well as reducing response times to users.
# Transaction guarantees (ACID)
**ACID** is the 4 guarantees for transactions:
- **Atomicity**: either all actions in a transaction occur, or none do
- **Consistency**: if each transaction is consistent, and the database starts in a consistent state, then the database will be consistent after the transactions are performed
- **Isolation**: if multiple transactions are executing simultaneously, then each individual transaction should should have the same effect as if it were executed alone
- **Durability**: if a transaction commits its effects persist
## Ensuring atomicity
There are two approaches:
1. **Write-ahead logs**: each action is written to a log, which are then executed on commit. This is used by almost all modern DBMSes, as it allows for transforming random writes into sequential writes, and can provide an audit trail of everything an app does.
2. **Shadow paging (copy on write)**: the DBMS makes copies of pages and transactions make changes to the copies. When a transaction completes, its shadow pages become visible to others.
## Ensuring isolation
A **concurrency control** protocol determines the proper interleaving of operations from multiple transactions. There are two main approaches:
1. **Pessimistic**: don't let problems arise in the first place, using locks or similar
2. **Optimistic**: assume conflicts are rare, and deal with them after they happen
### Example
If we have two accounts `A` and `B`, each with £1000, and `T1` transfers £100 from `A` to `B` and `T2` crediting both accounts with 6% interest, then there are many outcomes from running both simultaneously. There is no guarantee one will run before the other, but the net effect must be equivalent to them running serially in some order:
![[w6n3serialCorrect.png]]
These operations can also be interleaved, and as long as $A+B=2120$, everything is fine:
![[w6n3interleavedCorrect.png]]
There are however many interleavings that are not consistent:
![[w6n3interleavedIncorrect.png]]
### Schedule
A **schedule** is an ordering of reads and writes from a transaction. It is correct if it is equivalent to some serial execution of the transactions.

A schedule $S$ for a set of transactions $\{T_1,...,T_n\}$ contains all steps of all transactions and the ordering among steps in each $T_i$ is preserved, so $S=\langle R_1(B),R_2(A),W_2(B),W_1(A)\rangle$ where $R_1(A)$ is $T_1$ reading $A$.

Schedules are **equivalent** if for all database states, the effect of executing the first schedule is identical to the effect of executing the second schedule.

A schedule that does not interleave the actions of different transactions is **serial**:
![[w6n3serialSchedule.png]]

A schedule is **serialisable** if it is equivalent to some serial execution of the transactions. If each transaction preserves consistency, then every serialisable schedule preserves consistency.
#### Conflicting operations
Two operations conflict if the are by different transactions, on the same object, and at least one of them is a write. There are three types:
1. **Read-write (RW) conflicts**: consecutive reads in the same transaction must have the same result
   ![[w6n3rwConflict.png]]
2. **Write-read (WR) conflicts**: reading uncommitted data prevents rollbacks
   ![[w6n3wrConflict.png]]
3. **Write-write (WW) conflicts**: overwriting uncommitted data loses updates
   ![[w6n3wwConflict.png]]

