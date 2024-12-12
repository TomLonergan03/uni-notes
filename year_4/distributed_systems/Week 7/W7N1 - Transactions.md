A **transaction** is a series of operations executed by a client, which either **commits** (all operations succeed and server-side data is updated) or **aborts** (no operations occur and there is no effect on the server).
# ACID properties
All transactions adhere to the ACID principles:
- **Atomicity**: transactions either commit or abort
- **Consistency**: transactions do not violate system invariants
- **Isolation**: concurrent transactions do not interfere with each other
- **Persistence**: once a transaction commits, the changes are permanent

Ideally, we want to achieve all these while allowing for concurrency.
# Serial equivalence interleaving
If each of several transactions has the correct effect when done on their own, then we can infer that if the transactions are done one at a time in some order then the combined effect will also be correct. **Serial equivalence interleaving** is an interleaving of operations of transactions such that the combined effect is the same as if the transactions were performed sequentially in some order. Operations conflict for when one is a read and the other a write, or both are a write. 2 read operations do not conflict.
# Resolving conflicts
The naive approach is to check for serial equivalence between overlapping transactions at commit, and if there is a conflict abort the transactions. We can do better than this though, by preventing violations from occurring in the first place.

The pessimistic approach assumes transactions will conflict, so prevents transactions from accessing the same objects using [[W6N1 - Coordination|locks]]. This is better for when data is updated frequently, and can use reader-writer locks to improve performance by allowing readers to run concurrently with each other while writes get exclusive access to the object.

The optimistic approach assumes transactions won't conflict, so allows them to occur then checks later for conflicts. There are multiple ways to do this, including timestamp ordering and multi-version concurrency control.
# Distributed transactions
In a distributed transaction, there may be objects involved that reside on different servers. During a commit, the system needs to ensure all servers commit their corresponding update or if one server fails to commit then everyone aborts.
## Two phase commits
When the client commits, it sends a message to the leader. The leader then sends a `prepare` message to each server, which respond with `yes` or `no` depending on if they are able to commit, and if any `no`s are received then the leader tells all servers to abort, otherwise it tells them all to commit.