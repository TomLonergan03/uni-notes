# System architecture
There are three main system architectures that are used in practice
## Shared memory
![[w8n1sharedMemory.png]]
All CPUs share access to a common memory address space over a fast interconnect. This makes it efficient to send messages between processors, and all processors have a global view of all in-memory datastructures. Each DBMS instance on a processor has to know about the other instances. This architecture struggles to scale beyond a certain size once the interconnect bandwidth is saturated.
## Shared disk
![[w8n1sharedDisk.png]]
All CPUs can access a single logical disk directly via an interconnect, with each CPU maintaining its own private memory. This allows for scaling of the execution layer independently from the storage layer, and makes for easy consistency and [[W7N2 - Recovery|fault tolerance]] as there is a single copy of the database, thought the disk eventually becomes a bottleneck with too many CPUs.
## Shared nothing
![[w8n1sharedNothing.png]]
Under a **shared nothing** architecture, each DBMS instance has its own CPU, memory, and disk, typically running on commodity hardware. Nodes can only communicate between each other via the network, which makes it easy to increase capacity but hard to ensure consistency.
# Types of parallelism
**Inter-query parallelism** executes multiple queries concurrently, which increases throughput, reduces latency, and requires parallel-aware [[W7N1 - Locking|concurrency control]].

**Intra-query parallelism** executes the operations of a single query in parallel. This decreases latency for long-running queries. **Inter-operator parallelism** executes operators of a single query in parallel exploiting pipelining, while **Intra-operator parallelism** has all CPUs working on the same operation (e.g. scan, sort, join).
# Database partitioning
We can **partition** a database across multiple resources (nodes/disks/processors). This is also known as **sharding**. The DBMS will execute query fragments on each partition, when combine the results to produce a single answer.
## Horizontal partitioning
**Horizontal partitioning** splits a table's tuples into disjoint subsets by choosing a column/columns that divide the database equally in terms of size, load, or usage.
### Round robin partitioning
![[w8n1roundRobinPartitioning.png]]
**Round robin partitioning** distributes tuples in a round robin fashion, so the first tuple goes to the first partition, the second to the second, and so on. This spreads the load evenly, but it makes it hard to find which partition has which value.
### Hash partitioning
![[w8n1hashPartitioning.png]]
**Hash partitioning** uses a hash function on a column to determine what partition a tuple is in. This makes it easy to find the relevant partition for any constant value in a column, and is good for equi-joins and group by.
### Range partitioning
![[w8n1rangePartition.png]]
**Range partitioning** divides a column into ranges, then stores each range on a specific partition. This makes it easy to find the partition that hosts specific values, and is good for equi-joins, group-by, and range queries.
# Replication
The DBMS can replicate data across nodes to increase availability. **Partition replication** stores a copy of an entire partition in multiple locations, and **table replication** stores an entire copy of a table in multiple locations, though this is usually only used for small, read-only tables. The DBMS ensures updates correctly propagate to all replicas in either case.
## Data transparency
Users should not be required to know where data is physically located, or how tables are partitioned or replicated. An SQL query that works on a single node DBMS should work the same way on a distributed DBMS.
# Intra-operator parallelism
## Parallel scans
Files can be scanned in parallel, then concatenated to get an output. If data is partitioned using range or hash partitioning, and a selection predicate $p$, we can skip all partitions which contain no tuples satisfying $p$.
## Key lookup
if data is partitioned on a key, lookup requires querying only the partition the key is in. If not, then the lookup request must be broadcast to all nodes.
## Parallel hashing
Use a hash function $h_n$ to partition data over all node, then run [[W4N2 - Aggregation#External hashing|external hashing]] on each node independently.
![[w8n1parallelHashing.png]]
## Parallel hash join
Parallel hash joins can be performed by partitioning both relations on the join key, then performing a normal [[W4N3 - Joins#Hash joins|hash join]] on each node independently.
## Parallel sorting
Parallel sorting is achieved by range partitioning data over machines, then perform [[W4N1 - Sorting#External sorting|external sorting]] on each machine independently. [[W4N3 - Joins#Sort-merge join|Sort-merge joins]] can be performed similarly.
# Join scenarios
Distributed joins require having matching tuples on the same node. Once they are there we can use standard joining techniques. We will look at equi-joins here, as more complex joins either require streaming all data to every node or other more complex algorithms outside the scope of the course.
## Replicated small table
If one table is very small and regularly joined against, it can be replicated at every node then the results merged at a coordinating node.

E.g. for this query where `S` is small:
```sql
SELECT * FROM R JOIN S
ON R.id = S.id
```
![[w8n1joinScenario1.png]]
After this, $P1$ and $P2$ are concatenated to produce $R\bowtie S$.
## Same partitions
If both tables are partitioned on the join attribute, we can again perform the join on the local data then it is coalesced by a coordinator.

E.g. for the same query where `R` and `S` are partitioned on the join key:
![[w8n1joinScenario2.png]]
After this, $P1$ and $P2$ are concatenated to produce $R\bowtie S$.
## Different partition keys, one small table
If both tables are partitioned on different keys, but one of the tables is small, it can be broadcast to all nodes, which then perform the join locally.

E.g. for the same query where `S` is small:
![[w8n1joinScenario3.png]]
`S` is broadcast to both nodes, then the join is performed normally.
## Different partition keys, two large tables
If neither partition is small enough to broadcast or neither is partitioned on the join key, both of the tables are reshuffled across nodes, essentially repartitioning them on the join key.

E.g. for the same query:
![[w8n1joinScenario4.png]]
`R` and `S` are reshuffled across both nodes, then joined as usual.
# Query planning
Previous optimisations are still applicable in a distributed environment, such as predicate pushdown, early projection, and finding optimal join orders, but the DBMS must also consider the network cost of moving data between nodes. The network cost is likely to have a greater impact on performance than the number of IOs performed by a given node. In addition, nodes may need to wait for data from other nodes before they can start processing data, and we want to balance work between all nodes as evenly as possible, as a parallel query will take as long as the maximum runtime of any node in the system.
