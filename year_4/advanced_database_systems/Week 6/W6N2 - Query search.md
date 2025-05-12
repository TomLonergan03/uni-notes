There are two main cases for plan enumberation:
- Single-table plans (the base case)
- Multi-table plans (induction)

Single table queries include selects, projects, and aggregates.
# Single-table cost estimates
If we have an index $I$ on the primary key which matches selection, we will need to read $(Height(I)+1)+1$ pages for a variant B or C B+ tree.

If we have a range-style predicate, along with a clustered index $I$ matching the selection, we will need to read (approximately) $(NPages(I)+NPages(R))*selectivity$ pages.

If we have an non-clustered index for the same query, we have a cost of $(NPages(I)+NTuples(R)))*selectivity$ pages.

A sequential scan of a file has a cost of $NPages(R)$.
## Example
If we have a query:
```sql
SELECT * FROM Sailors WHERE rating = 8
```
And know:
- $NTuples(Sailors)=40,000$
- $NPages(Sailors)=500$
- $NKeys(rating)=10$
- $NPages(I)=50$

If we have an index $I$ on $rating$, we have an expected output cardinality of $1/NKeys(rating)\cdot NTuples(Sailors)=1/10\cdot40,000=4,000\text{ tuples}$. A clustered index will read $1/NKeys(rating)\cdot(NPages(I)+NPages(Sailors))=1/10\cdot(50+500)=55\text{ pages}$, while an unclustered index will read $1/NKeys(rating)\cdot(NPages(I)+NTuples(Sailors))=1/10\cdot(50+40,000)=40,050\text{ pages}$.

If we have an index $I$ on $sid$, doing an index scan retrieves all pages and tuples, with a clustered index reading $\approx(50+500)$ pages and an unclustered index reading $\approx(50+40,000)$ pages.

Doing a file scan reads all 500 file pages.
# Multi-table plans
Most mainstream DBMSes only support 2-way [[W4N3 - Joins|joins]], which very quickly leads to enormous search spaces:

| Number of relations $n$ | Number of different join trees |
| ----------------------- | ------------------------------ |
| 2                       | 2                              |
| 3                       | 12                             |
| 4                       | 120                            |
| 5                       | 1680                           |
| 6                       | 30,240                         |
| 7                       | 665,280                        |
| 8                       | 17,297,280                     |
| 10                      | 17,643,225,600                 |
This means that we cannot enumerate all possible join trees for any real query, so the search space must be restricted.

One approach (used in System R in the 70s) is to only consider left-deep join trees:
![[w6n2joinTrees.png]]

Modern DBMSes do often prefer left-deep join trees, as this results in the inner (right) relation is always a base relation, which allows the use of index nested loop joins, along with fully pipelined plans where intermediate results are not written to temporary files (though only if all operators are non-blocking). Modern systems will often consider non left-deep join trees in addition.

Focusing on left-deep trees only, we can enumerate left-deep trees, and eliminate cross products immediately. We can then enumerate the plans for each operator, and the access paths for each table. We can use dynamic programming to reduce the number of cost estimations, as the best left-deep plan to join tables $R,S,T$ is either $(\text{the best plan for joining R and S})\bowtie T$, $(\text{the best plan for joining R and T})\bowtie S$, or $(\text{the best plan for joining S and T})\bowtie R$/
## Example
For a query:
```sql
SELECT * FROM R, S, T
WHERE R.A = S.A
AND S.B = T.B
```
On the first pass, we enumerate all access paths for each relation (index vs full table scans).
On pass 2, we find the best 2-relation plans by looking at each join order:
![[w6n2pass2candidates.png]]
then determine the best candidate:
![[w6n2pass2best.png]]

On pass 3, we enumerate the best 3 relation paths using the best 2-relation plans and one other relation:
![[w6n2pass3candidates.png]]
and can select the overall best plan:
![[w6n2pass3best.png]]
## Interesting orders
We may also consider **interesting orders**, sorted orders of input tables which may be beneficial later in the query plan, e.g. a later step may perform a sort-merge join cheaply if its input is already sorted. This can be done by retaining both the cheapest plan overall, and the cheapest plan for each interesting order of the tuples.
### Example
For a plan:
```sql
SELECT S.sid, COUNT(*) AS number
FROM Sailors S
JOIN Reserves R ON S.sid = R.sid
JOIN Boats B ON R.bid = B.bid
WHERE B.colour = ‘red’
GROUP BY S.sid
```
with indexes:
- `Sailors`: B+ tree on `sid`
- `Reserves`: Clustered B+ tree on `bid`, B+ tree on `sid`
- `Boats`: B+ tree on `colour`

On pass 1, find the best plan for each relation and for interesting orders:
- `Sailors`, `Reserves`: file scan
- `Boats`: B+ tree on colour
- B+ tree on `Sailors.sid` as interesting order (output sorted on `sid`)
- B+ tree on `Reserves.bid` as interesting order (output sorted on `bid`)
- B+ tree on `Reserves.sid` as interesting order (output sorted on `sid`)

On pass 2, find best 2 relation plans:
```
// for each left-deep logical plan
foreach plan P in Pass 1:
	foreach FROM table T not in P:
		// for each physical plan
		foreach access method M on T:
			foreach join method ⨝:
				generate P ⨝ M(T)
```
We eliminate cross products, and retain the cheapest plan for each (pair of relations, order).

On pass 3, use pass 2 plans as outer relations and generate plans for the next join in the same way as pass 2.

Finally, we add the cost for group-by/aggregate for all plans that don't already sort the result by `sid`, and choose the cheapest plan.