Sorting is used for many different query operations, such as:
- Explicit sorting with `ORDER BY`
- Implicit sorting for duplicate elimination with `DISTINCT`
- Implicit sorting to prepare for equi-joins
- Implicit in `GROUP BY`
- First step for building a [[W3N3 - Tree-based indexing#B+ tree|B+ tree]] is to sort the leaf entries

A file is sorted with respect to key $k$ and ordering $\Theta$ if for any two adjacent records $r_1$ and $r_2$ where $r_1$ precedes $r_2$ in the file, their corresponding keys are in $\Theta$ order:
$$
r_1\ \Theta\ r_2\Leftrightarrow r_1.k\ \Theta\ r_2.k
$$
A key may be a single attribute or an ordered list of attributes. In the latter case we sort by a key whenever all previous keys are equal on both records, e.g. for key $(A,B)$ and $\Theta$ is $<$:
$$
r_1<r_2\Leftrightarrow r_1.A<r_2.A\ \vee\ (r_1.A=r_2.A\ \wedge\ r_1.B<r_2.B)
$$

If the data to be sorted fits in memory, then we can use a standard sorting algorithm like quicksort. If the data does not fit in memory, then we need to use a technique that is aware of the cost of writing data to disk.
# External sorting
**External sorting** is used to sort a file that is larger than the available main memory space. It uses divide and conquer to sort chunks of data that fit in memory, then write back the sorted chunks, and combines them into a single larger file.
## 2-way external merge sort
![[w4n1twoWayExternalMerge.png]]
We can sort any amount of data using only 3 buffer pages, building up sorted sets of pages called runs:
- On the first pass, each page is read into memory, sorted, and written back to disk
- On each subsequent pass, pairs of runs are merged to produce a run of double the length. When either input page is consumed, read the next page of that run from disk, and write the output page to disk each time it is full.

This requires $1+\lceil\log_2N\rceil$ passes, with each page being read and written once per pass, resulting in a total IO cost of $2N\cdot\text{Number of passes}=2N\cdot(1+\lceil\log_2N\rceil)$ operations.
## N-way external merge sort
![[w4n1nWayExternalMerge.png]]
With more buffer pages ($B$), we can sort $B$ pages simultaneously in memory, and we can merge $B-1$ runs per pass. This will produce $\lceil N/B\rceil$ sorted runs of size $B$ on the initial pass, then merge on each subsequent run, resulting in $1+\lceil\log_{B-1}\lceil N/B\rceil\rceil$ passes for a total IO cost of $2N\cdot(1+\lceil\log_{B-1}\lceil N/B\rceil\rceil)$ IO operations.

E.g. if we have $N=108$ pages in a file and $B=5$ buffer pages, then on:
- Pass 0 there are $\lceil108/5\rceil=22$ sorted runs of 5 pages each, with the last run only having 3 pages
- Pass 1 there are $\lceil22/4\rceil=6$ sorted runs of 20 pages each, with the last run only having 8 pages
- Pass 2 there are $\lceil6/4\rceil=2$ sorted runs, one with 80 pages and the other with 28
- Pass 3 results in a sorted file of 108 pages

This gives us:
$$
\begin{aligned}
\text{Number of passes }&=1+\lceil\log_{B-1}\lceil N/B\rceil\rceil\\
&=1+\lceil\log_422\rceil\\
&=1+\lceil2.229...\rceil\\
&=4\\
\\
\text{Total IO cost }&=2N\cdot\text{\# of passes}\\
&=2\cdot108\cdot4\\
&=864
\end{aligned}
$$
# Using B+ trees for sorting
If we have a [[W3N2 - Indexes#Clustered vs unclustered indexes|clustered]] [[W3N3 - Tree-based indexing#B+ tree|B+ tree]], then we can traverse to the leftmost leaf page, and either read all data from the leaf pages (when using [[W3N2 - Indexes#^055a12|variant A]] indexes), or follow the pointers to each page once (for [[W3N2 - Indexes#^c4b2e9|variant B]] indexes). This is always faster than external sorting, but requires an existing index on the sort values.

If we have an unclustered B+ tree, then we have to follow each individual record pointer to the page that contains that pointer, which results in 1 IO per record, so it is almost always a bad idea to use an unclustered B+ tree for sorting
# Duplicate elimination using sorting
If we sort a table, filter it, and remove the columns we don't need, then if we sort the table we only need to check adjacent records to eliminate duplicates.

```SQL
SELECT DISTINCT cid FROM Enrolled WHERE grade < 90
```
`Enrolled(sid, cid, grade)`

| sid    | cid        | grade |
| ------ | ---------- | ----- |
| 123466 | INFR-11011 | 65    |
| 123488 | INFR-11122 | 95    |
| 123488 | INFR-10070 | 80    |
| 123466 | INFR-11122 | 70    |
| 123455 | INFR-11011 | 75    |
filtered:

| sid    | cid        | grade |
| ------ | ---------- | ----- |
| 123466 | INFR-11011 | 65    |
| 123488 | INFR-10070 | 80    |
| 123466 | INFR-11122 | 70    |
| 123455 | INFR-11011 | 75    |
removing columns:

| cid        |
| ---------- |
| INFR-11011 |
| INFR-10070 |
| INFR-11122 |
| INFR-11011 |
sorted:

| cid        |
| ---------- |
| INFR-10070 |
| INFR-11011 |
| INFR-11011 |
| INFR-11122 |
then we can remove duplicates:

| cid        |
| ---------- |
| INFR-10070 |
| INFR-11011 |
| INFR-11122 |
