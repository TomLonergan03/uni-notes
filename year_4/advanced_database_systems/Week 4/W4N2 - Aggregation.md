If we don't need ordered data, we can use hashing instead of [[W4N1 - Sorting|sorting]] to form groups (`GROUP BY`) and to remove duplicates (`DISTINCT`).
# External hashing
We can't build an in-memory hash table if there is too much data, so we start by splitting it into smaller pieces using a hash function $h_p$. If we have $B$ pages of buffer we can split the data into $B-1$ partitions.
![[w4n2externalHashingPartition.png]]
If each partition is small enough to fit in memory, we can load them in and make an in-memory hash table for each one, one at a time, and apply duplicate removal/aggregation. As every tuple in a partition has the same value when $h_p$ is applied, the in memory hash table must use a different hash function $h_f$ that is independent of $h_p$.
![[w4n2externalHashingRehashing.png]]
# Aggregations
Aggregates collapse multiple tuples into a single scalar value. We do this using hashing, and then populate a temporary in memory hash table. For each record, check if there is an entry in the hash table. For `DISTINCT`, if there is already an entry the current record can be discarded, and for `GROUP BY` the aggregate computation is performed with the result combined with the result already stored in the hash table.

If the resulting hash table does not fit in memory, then we use external hashing, build the aggregate for each partition, then combine those results.
# Cost
We can hash a $B\cdot(B-1)$ table, as there are $B-1$ partitions created in the first stage of the hashing process, and each resulting partition must be able to entirely fit in the buffer so is $B$ pages.
# Sorting vs hashing
An external merge sort will often finish 2-3 passes, so is better if we need the output to be sorted anyway or want to preserve duplicates. Hashing is preferred for duplicate elimination as it scales with the number of distinct values (unlike sort which scales with the number of values), and aggregation is normally computed using hashing.