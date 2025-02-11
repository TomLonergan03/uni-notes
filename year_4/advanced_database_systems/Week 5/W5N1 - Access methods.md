# Query plan
The operators from [[W1N3 - Relational algebra|relational algebra]] are arranged into a tree called the **query plan**. The edges in the tree indicate data flow between operators, with data flowing from the leaves towards the root. The output of the root is the query result.

e.g.
```SQL
SELECT R.id, S.value
FROM R, S
WHERE R.id = S.id
AND S.value > 100
```
forms the tree:
![[w5n1queryPlan.png]]
## Query operators
For a given relational algebra operator $o$, a typical DBMS query engine may provide different implementations ($o^\prime,o^{\prime\prime},...$), called physical operators, each semantically equivalent to $o$. Physical operators exploit properties such as the presence or absence of indexes, sortedness and size of inputs, space in the buffer pool, buffer replacement policy, etc.
# Query evaluation workflows
Queries are evaluated by:
1. Parsing the query
2. Translating the query to relational algebra
3. Enumerate plans by selecting physical operators and orders of operators
4. Estimating the cost of each plan
5. Selecting the optimal plan (or approximating optimal, as the space of possible plans is usually far too large to exhaustively search)
# Access methods
An **access method** is a way the DBMS can access the data in a table. It is not defined in relational algebra, but includes selection predicates. There are 3 basic approaches:
- sequential scans
- [[W3N2 - Indexes|index]] scans
- multi-index/bitmap scan
## Sequential scan
```python
for page in table.pages:
	for t in page.tuples:
		if evalPred(p, t):
			// do something
```
Iterate through all tuples in each page in the table, and check if the tuple matches some predicate $p$. The IO cost of a sequential scan is $N$ page reads, and outputs $sel(p)\cdot N$ pages, where $sel(p)$ is the selectivity of the predicate (the fraction of tuples satisfying predicate $p$).
## Index scan
The DBMS picks an index to find the tuples that the query needs. Which index is used (and whether one is at all) is dependent on:
- What attributes the index contains
- What attributes the query references
- Whether the index has unique or non-unique keys
- Whether the index is clustered or unclustered
### Composite search keys
A tree index with a **composite search key** on columns $(A_1,A_2,...,A_n)$ matches a predicate if the predicate is a conjunction of $0\leq m\leq n$ equality clauses of the form $A_1=c_1\text{ AND }A_2=c_2\text{ AND }...\text{ AND }A_m=c_m$, and at most one additional range clause of the form $\text{AND }A_{m+1}\ op\ c_{m+1}$ where $op$ is one of $<,>,\leq,\geq,\text{ BETWEEN}$. This is as we lookup the first tuple in lexicographic order, then scan until the range clause no longer holds.

E.g. if we have a tree index on $(Age, Salary)$, then $Age=31\text{ AND }Salary=400$, $Age=31\text{ AND }Salary>200$, and $Age>31$ are all lexicographic ranges, but $Age>31\text{ AND }Salary=400$ is not lexicographic, and neither is $Salary=300$.
### Index-only scan
It may be possible to answer a query without retrieving any tuples from some tables, e.g. with an index on `E.dno`, the query
```SQL
SELECT E.dno, COUNT(*) FROM Employee E GROUP BY E.dno
```
This does not need to load any value of `E.dno` from the heap pages, as they will be in the index as keys.

These kinds of searches are often much faster than heap scans due to small index sizes.
#### Clustered B+ tree scan
A clustered B+ tree index whose search key matches the selection predicate $p$ is generally cheap, with an IO cost of $2-4$ pages to reach a leaf page, then $sel(p)\cdot\text{number of leaf pages}$. If we use variants B or C, then we may also need to access data records (but only if we are using non-search-key attributes) for an additional cost of $sel(p)\cdot\text{number of data pages}$.
#### Unclustered B+ tree scan
An unclustered B+ tree is much less efficient to scan, as each leaf entry points to a different page for an IO cost of the number of leaf index entries. Index only scans are as fast as on clustered B+ trees. If $sel(p)$ is close to 0, lack of clustering is a minor issue.
#### Hash index scan
A hash index matches a selection predicate $p$ only if $p$ contains a term of the form $A=c$ and the hash index is built over column $A$. Composite search keys must be bounded entirely i.e. a hash index on $(age,dept)$ matches $age=27\text{ AND }dept=CS$ but does not match $age=27$.
## Multi-index scan
If there are multiple indexes on the same table, the DBMS may use both indexes for a query by computing sets of record IDs for each index, then combining them based on the query's predicates (either union or intersect). Set intersection can be done with bitmaps, hash tables, or Bloom filters.

E.g. If we have a table with two indexes:
- Tree index 1 on $age$
- Index 2 on $dept$

and we run the query
```sql
SELECT * FROM Students WHERE age < 30 AND dept = 'CS' AND country = 'UK'
```
then the DBMS may use both indexes:
6. Retrieve the record ids satisfying $age<30$ using tree index 1
7. Retrieve the record ids satisfying $dept=\text{`CS'}$
8. Take their intersection
9. Retrieve the records and check $country=\text{`UK'}$

