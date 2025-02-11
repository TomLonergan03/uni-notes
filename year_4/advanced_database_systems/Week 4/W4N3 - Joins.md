The join operation $\bowtie$ will concatenate tuples from two relations together when a join attribute applies, e.g. 
```SQL
SELECT R.id, S.city
FROM R, S
WHERE R.id = S.id
AND S.value > 100
```
will perform a join of $\bowtie_{R.id=S.id}$. After the join subsequent operators will never need to go back to the base tables to get more data.

Joins are the most expensive operator that exist database systems, with the number of joins often being used as a measure of query complexity.

A naive implementation of joins would be $R\bowtie_cS\equiv\sigma_c(R\times S)$, performing a cross product then filtering on the join condition, but this is inefficient as the cross product is very large (equal to $|R|\cdot|S|$).

There are three classes of join algorithms, with none working well in all situations:
- Nested loops
- Sort-merge
- Hash
# IO cost analysis
We will assume table $R$ has $M$ pages with $m$ tuples total, and table $S$ has $N$ pages and $n$ tuples total. The cost metric we will use is the number of IO operations required to compute the join, ignoring output costs and CPU costs.
# Nested loop joins
## Simple nested loop join
```
foreach tuple r in R:
	foreach tuple s in S:
		emit if r and s match
```
This is slow as it scans $S$ once for each tuple in $R$. This is very slow if $S$ doesn't fit in memory.

This has a total cost of $M+m\cdot N$.

On an example database with $M=1000,m=100,000,$$N=500,n=40,000$ we get an overall cost of $1000+100,000\cdot500=50,001,000\text{ IOs}$. At 0.1ms per IO, this results in a total time $\approx1.4\text{ hours}$.
## Page nested loop joins
If we explicitly write out page fetches, the simple loop is equivalent to:
```
foreach page pR in R:
	foreach tuple r in pR:
		foreach page pS in S:
			foreach tuple s in pS:
				emit if r and s match
```
If we flip the r tuple and PS page loops, we get 
```
foreach page pR in R:
	foreach page pS in S:
		foreach tuple r in pR:
			foreach tuple s in pS:
				emit if r and s match
```
This makes far fewer disk accesses, as for each *page* in $R$, we scan $S$ once. This has a cost of $M+M\cdot N$. We also want to ensure the outer table is the smaller table in number of pages

On the same example database as above, we get an overall cost of $N+N\cdot M=500+1000\cdot500=500,500\text{ IOs}$, for a total time $\approx50\text{ seconds}$. This requires only 3 buffer pages, one for each input and one for the output.
## Block nested loop joins
If we have $B$ buffer pages available, we can have more of the outer table in memory at once. We use $B-2$ buffers for scanning the outer table, 1 buffer for scanning the inner table, and 1 buffer for storing the output.
```
foreach B-2 block bR in R:
	foreach block bS in S:
		foreach tuple r in bR:
			foreach tuple s in bS:
				emit if r and s match
```
This requires one scan of the outer table, and $M/B-2$ scans of the inner table, for a cost of $M+\lceil M/(B-2)\rceil\cdot N$. If the outer relation $R$ fits in memory (so $M\leq B-2$), we get a cost of $M+N=1000+500=1500\text{ IOs}$, or $\approx0.15\text{ seconds}$.
## Index nested loop joins
If we have an [[W3N2 - Indexes|index]] on the inner loop join condition, we can use it to avoid iterating all tuples.
```
foreach tuple r in R:
	foreach tuple s in Index(r = s)
		emit if r and s match
```
This has a cost of $M+m\cdot\text{cost to find all matching S tuples}$. The index access cost per $R$ tuple is:
- [[W3N3 - Tree-based indexing#B+ tree|B+ tree]] is 2-4 IOs to reach a leaf and fetch matching $S$ tuples. For a clustered tree, this costs a total of $M+m\cdot\text{search + number of matching pages}$ while for an unclustered tree it costs $M+m\cdot\text{search + number of matching tuples}$
- [[W3N4 - Hash-based indexing|Hash index]] uses 1-2 IOs to reach the target bucket
# Sort-merge join
If a join uses an equality predicate, we can use a **sort-merge join**. This has two steps:
1. **Sort** both tables on the join key(s), e.g. using [[W4N1 - Sorting#N-way external merge sort|external merge sort]]. This may not be necessary if a previous operation has already produced a sorted output, either intentionally or as a byproduct of the operation it was performing.
2. **Merge**: scan both tables in parallel and emit matching tuples

```
sort R, S on join key A
r = position of first tuple in Rsorted
s = position of first tuple in Ssorted
while r != EOF and s != EOF:
	if r.A > s.A:
		advance s
	else if r.A < s.A:
		advance r
	else: // r.A must equal s.A
		emit (r, s)
		advance s
```

This has a cost of:
$$
\begin{aligned}
\text{Sort cost }R&=2M\cdot(1+\lceil\log_{B-1}\lceil M/B\rceil\rceil)\\
\text{Sort cost }S&=2N\cdot(1+\lceil\log_{B-1}\lceil N/B\rceil\rceil)\\
\text{Merge cost}&=M+N\\
\text{Total cost}&=\text{Sort cost }R+\text{Sort cost }S+\text{Merge cost}
\end{aligned}
$$

Repeating on the same tables we have used and with 100 buffer pages:
$$
\begin{aligned}
\text{Sort cost }R&=2\cdot1000\cdot(1+\lceil\log_{99}\lceil 1000/100\rceil\rceil)\\
&=2\cdot1000\cdot2\\
&=4000\text{ IOs}\\
\text{Sort cost }S&=2\cdot500\cdot(1+\lceil\log_{99}\lceil 500/100\rceil\rceil)\\
&=2\cdot500\cdot2\\
&=2000\text{ IOs}\\
\text{Merge cost}&=1000+500\\
&=1500\text{ IOs}\\
\text{Total cost}&=4000+2000+1500\\
&=7500\text{ IOs}
\end{aligned}
$$
which results in a time of $\approx0.75\text{ seconds}$.

We can refine sort-merge join by combining the last pass of merge-sort with the merge phase of join if the number of sorted runs is at most $B-1$. This saves 1 full read and write of $R$ and $S$, for a total cost of $2000+1000+1500=4500\text{ IOs}$, or a runtime of $\approx0.45\text{ seconds}$.

Sort merge join is useful when one or both tables are already sorted on the join key, when the output must be sorted on the join key, or when working with very large datasets as it uses sequential access very effectively.
# Hash joins
If we are using an equality join predicate, we can use a hash table.
## Basic in-memory hash join
If one of the tables fits fully in memory, we can use two steps to join the tables:
1. **Build** a hash table using a hash function $h$ on the join key in the outer relation, and store the value in the hash table
2. **Probe**: scan the inner relation and use $h$ on each tuple to jump to a location in the hash table, which will contain all matching tuples.

```
foreach tuple r in R:
	add r to HTr using h(r)
foreach tuple s in S:
	emit if h(s) in HTr
```
This has a cost of $M+N$.
## Grace hash join
If neither relation fits in memory, we can decompose each relation into smaller partial joins, then run an in-memory hash join on each of those partial joins. This is the **Grace hash join**.
![[w4n3graceHashJoin.png]]
Here, each partition contains possible joins.

```
Rpt = partition R using h
Spt = partition S using h
foreach partition i:
	HTri = build hash table for R[i]
	foreach tuple s in S[i]:
		emit if h(s) in HTri
```

If partitions do not split in memory, we can use another hash function to further partition each partition.

If we have enough buffers to fit each pair of partitions we can read all and write pages and write both tables in $2(M+N)\text{ IOs}$, then in the build and probe phase we read both tables in $M+N\text{ IOs}$, for a total cost of $3(M+N)\text{ IOs}$.

On our example database, we will have a cost of $3(1000+500)=4500\text{ IOs}$, for a time of $\approx0.45\text{ seconds}$.
## Hash vs sort-merge joins
Sorting is good if input is already sorted, or the output needs to be sorted in a later step. It is not sensitive to data skew or bad hash functions.

Hashing is good as the number of passes depends on the size of the smaller relation, so if it is $<B$ then in-memory is great. It is useful if the input is already hashed or the output needs to be hashed.
# Summary of performance
| Join algorithm                                          | IO cost                         | Total time                      |
| ------------------------------------------------------- | ------------------------------- | ------------------------------- |
| Simple nested loop join                                 | $M+m\cdot N$                    | 1.4 hours                       |
| Page nested loop join                                   | $M+M\cdot N$                    | 50 seconds                      |
| Block nested loop join                                  | $M+\lceil M/(B-2)\rceil\cdot N$ | varies depending on $B$         |
| Index nested loop join                                  | $M+m\cdot\text{access cost}$    | varies depending on access cost |
| Sort-merge join                                         | $M+N+\text{sort cost}$          | 0.75 seconds                    |
| Hash join                                               | $3(M+N)$                        | 0.45 seconds                    |
| Nested loop or hash (if<br>one relation fits in memory) | $M+N$                           | 0.15 seconds                    |
