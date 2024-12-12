Replicating data at one or more sites can help with:
- Availability and fault tolerance, as if a primary server crashes a secondary server can take over
- Performance: servers can be located near clients, reducing latency, and concurrent reads can be served from multiple servers
- Scaling: adding additional servers can help to manage increased usage
# Challenges
Once there are multiple copies of data, those copies must be kept consistent. Synchronising replicas is a challenging problem.
# Performance and scalability
To keep replicas consistent, we generally need to ensure that all conflicting operations are done in the same order across all servers. Guaranteeing this may be costly, so one solution is to weaken consistency requirements wherever possible, but this is not possible in all cases e.g. Facebook can have a delay on synchronising like counts, but a stock market must update all servers immediately no matter what.
# Consistency models
A **consistency model** is a contract between the programmer and a system. The system guarantees that if the programmer follows the rules for operating on data, the data will be correct. There are two consistency models:
- **Data-centric consistency models**: defines consistency as experienced by all clients with a system wide consistent view on the data store
- **Client-centric consistency models**: defines consistency only from each client's perspective, so different clients may see different sequences of operations at their replicas
# Distributed data stores
A **distributed data store** has data physically distributed and replicated across multiple machines. Data can be read or written by any process on any node, with a local copy to help with faster reads. A write to a local replica must be propagated to all remote replicas.
## Notation
- $W_i(x)a$: process $P_i$ writes value $a$ to $x$
- $R_i(x)b$: process $P_i$ reads value $b$ from $x$
- All data items initially have value $NIL$

Behaviour is represented over time which moves from left to right:
![[w7n2dataStoreModel.png]]
## Strict consistency
Under **strict consistency**, all writes are instantaneously visible to all processes. In practice this is not possible as messages take time to travel.
![[w7n2strictConsistency.png]]
## Sequential consistency
Under **sequential consistency**, all processes must see the same ordering of reads and writes. Any valid interleaving of read and write operations is fine, but all processes must see the same ordering.
![[w7n2sequentialConsistency.png]]
## Linearisability
**Linearisability** means that every operation must appear to take effect instantaneously at some moment between its start and completion. A data store is said to be linearisable when each operation is timestamped, sequential consistency holds, and if $timestamp(op_1(x))<timestamp(op_2(x))$ then $op_1(x)$ should precede $op_2(x)$.
### Consistency vs linearisability
Linearisability is weaker than strict consistency, but stronger than sequential consistency.
## Causal consistency
**Causal consistency** has writes that are causally related must be seen by all processes in the same order e.g. if event $A$ is a direct or indirect result of event $B$, then all processes should observe event $A$ before event $B$.
![[w7n2causalConsistency.png]]
## FIFO consistency
**FIFO consistency** has writes performed by a single process appear to all other processes in the order in which they were issued. Writes from different processes may be out of order. It is an easy form of consistency to implement.
![[w7n2fifoConsistency.png]]
