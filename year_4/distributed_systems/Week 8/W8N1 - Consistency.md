# Weak data-centric consistency models
We have seen [[W7N2 - Replication#Consistency models|strong data-centric consistency models]], but there are also weak consistency models where data, such as:
- **Weak consistency**: shared data is only reliably consistent after a synchronisation is done.
- **Release consistency**: shared data is made consistent when a critical region is exited
- **Entry consistency**: shared data pertaining to a critical region is made consistent when that critical region is entered

The weaker a consistency model is, the more scalable it is, as less time is needed to synchronise nodes.
# Client-centric consistency models
There are 6 main guarantees we can provide for client-centric consistency:
- **Strong consistency**: all readers will see all previous writes, and all readers see the same data
- **Eventual consistency**: Readers can see an arbitrary subset of previous writes, and eventually will see all writes
- **Consistent prefix**: The reader is guaranteed to see an ordered sequence of writes starting with the first write to a data object. This means the reader sees a version of the data store that existed at the primary node at some time in the past
- **Bounded staleness**: The reader sees all "old" writes. This guarantees that the reader will see all values written more than $T$ minutes ago, and an arbitrary subset of values written since then
- **Monotonic reads**: The reader is guaranteed to observe a data store that is increasingly up-to-date over time. If the reader issues a read operation, then later issues another, the second read will either return the same values as the first, or those values plus more from newer writes
- **Read my writes**: A client will see all values it has written, and an arbitrary subset of values written by other clients
## CAP trade-offs
We have to trade off between 3 properties: consistency, availability, and performance. This goes (roughly) as follows:

| Model                | Consistency | Performance | Availability |
| -------------------- | ----------- | ----------- | ------------ |
| Strong consistency   | Excellent   | Poor        | Poor         |
| Eventual consistency | Poor        | Excellent   | Excellent    |
| Consistent prefix    | Okay        | Good        | Excellent    |
| Bounded staleness    | Good        | Okay        | Poor         |
| Monotonic reads      | Okay        | Good        | Good         |
| Read my writes       | Okay        | Okay        | Okay         |
