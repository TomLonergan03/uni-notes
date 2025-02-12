The goal of query optimisation is to find a correct [[W5N1 - Access methods#Query plan|query plan]] that has the lowest cost. This is the hardest part of a DBMS to implement well, and is proven to be NP-hard so no optimiser produces the optimal plan and must use heuristics to limit the search space. At minimum we want to avoid really bad plans.

We will focus on IBM's System R optimisers, though other optimisation strategies exist.
# Query lifecycle
![[w5n2queryLifecycle.png]]
The **parser** checks correctness and authorisation, and generates a [[W1N3 - Relational algebra|RA]] parse tree. 
## Rewriter
The **rewriter** uses rules and heuristics to rewrite the query using rules and heuristics.

Some simplifications we could apply:
- Remove unnecessary predicates, e.g. `SELECT * FROM R WHERE 1 = 0` can be simplified to an empty result, while `SELECT * FROM R WHERE 1 = 1` simplifies to `SELECT * FROM R`
- Remove unnecessary joins, e.g. `SELECT R1.* FROM R AS R1 JOIN R AS R2 ON R1.id = R2.id` becomes `SELECT * FROM R`
- Ignore unnecessary nested subqueries, e.g. `SELECT * FROM R AS R1 WHERE EXISTS (SELECT * FROM R AS R2 WHERE R1.id = R2.id)` simplifies to `SELECT * FROM R`
- Merging predicates, e.g. `SELECT * FROM R WHERE val BETWEEN 1 AND 100 OR val BETWEEN 50 AND 150` simplifies to `SELECT * FROM R WHERE val BETWEEN 1 AND 150`
## Query optimiser
![[w5n2queryOptimiser.png]]
The query optimiser optimises one query block at a time. It enumerates all possible plans, or at least "promising" plan candidates. It determines the cost of each plan using a cost model and statistics from the catalogue, and chooses the best plan it finds per query block.

There are three components of query optimisation:
- **Plan space**: what plans are considered for a given query. A larger plan space means its more likely to find a cheaper plan, but will be harder to search
- **Cost optimisation**: how the cost of a plan is estimated
- **Search strategy**: the approach used to search the plan space
### Plan space
To generate a plan space, we need to know how to rewrite relational algebra expressions using equivalence rules and heuristics, and using equivalent physical operations
#### Relational algebra equivalences
$$
\begin{aligned}
\textbf{Selections}&\\
\sigma_{c_1\wedge c_2\wedge...\wedge c_n}(R)&\equiv\sigma_{c1}(\sigma_{c2}(...(\sigma_{c1})))&\text{(Cascade)}\\
\sigma_{c1}(\sigma_{c2}(R))&\equiv\sigma_{c2}(\sigma_{c1}(R))&\text{(Commute)}\\
\textbf{Projections}&\\
\pi_{a1}(...(R)...)&\equiv\pi_{a1}(...(\pi_{a1,...,an-1}(R))...)&\text{(Cascade)}\\
&\text{We can do partial projections as}\\
&\text{long as we keep everything we need}\\
\textbf{Cartesian products}\\
R\times(S\times T)&\equiv(R\times S)\times T&\text{(Associative)}\\
R\times S&\equiv S\times R&\text{(Communtative)}
\end{aligned}
$$
**Joins** are associative and commutative, but as they are Cartesian products with selections they can turn into cross-products, e.g. $(S\bowtie_{S.B=T.B}T)\bowtie_{S.A=R.A}R\equiv S\bowtie_{S.B=T.B\wedge S.A=R.A}(T\times R)$, and some join orders have cross products while others don't.

E.g. for the query
```sql
SELECT *
FROM R, S, T
WHERE R.A = S.A
AND S.B = T.B
```
We can make any of these join patterns:
![[w5n2joinOrderings.png]]
It is also possible to find implicit joins through transitivity, e.g. the above query is equivalent to 
```sql
SELECT *
FROM R, S, T
WHERE R.A = S.A
AND S.B = T.B
AND R.A = T.C
```
which makes the join ordering $(R\bowtie T)\bowtie S$ possible avoiding a Cartesian product.
#### Common heuristics
##### Selections
Generally we want to filter as early as possible, reorder predicates so that the most selective one is applied first, break complex predicates up and cascade them down, and simplify complex predicates (e.g. `X = Y AND Y = 3` can become `X = 3 AND Y = 3`, or `L.TAX * 100 < 5` can become `L.TAX < 0.05`).

E.g. we can simplify this expression:
![[w5n2selectionPushdown.png]]
This is an improvement as selection is essentially free while joins are expensive, so by selecting first we reduce the number of inputs to the join.
##### Projections
We want to perform them early to reduce intermediate results (if duplicates are eliminated), and to project out all attributes except the one requested or required (e.g. join keys). This is only a benefit for row stores, not for column stores.

E.g. here
![[w5n2projectionPushdown.png]]
we only need `sid` and `sname` columns, so we can project down to those columns before performing the join.
##### Joins
We want to avoid Cartesian products wherever possible, by using theta-joins instead.

E.g. favour $(R\bowtie S)\bowtie T$ over $(R\times T)\bowtie S$ in most cases, though if $R\times T$ is very small relative to $S$ it may be preferable to use the Cartesian product.
#### Physical equivalences
We can use a number of different physical approaches for the same operations:
- Base table accesses can be heap or index scans (if an index is available on the correct columns)
- Equi-joins can use a number of [[W4N3 - Joins|approaches]]
- Non-equi-joins must use block nested loops
# Example optimisation
If we have a database with the tables:
`Reserves(sid, bid, day, rname)` with 1000 pages, 100 tuples per page, 40 bytes per tuple, and `bid` is an equal distribution of 100 values.
`Sailors(sid, sname, rating, age)` with 500 pages, 80 tuples per page, 50 bytes per tuple, and `rating` is an equal distribution of 10 values.
We assume we have $B=5$ pages for use in joins, and our cost model is just counting IOs.

Our query is:
```sql
SELECT S.sname
FROM Reserves R, Sailors S
WHERE R.sid = S.sid
AND R.bid = 100
AND S.rating > 5
```

## Plan 1
We could do:
![[w5n2plan1.png]]
This costs 500 IOs to scan `Sailors`, then scans `Reserves` once per page of `Sailors` for 1000 IOs, for a total cost $=500+500\cdot1000=500,500\text{ IOs}$. This query works, but misses several opportunities.
## Plan 2
We can push the rating selection down to before the join. 
![[w5n2plan2.png]]
As half of sailors have a rating $>5$, we get a cost $=500+500/2\cdot1000=250,500\text{ IOs}$.
## Plan 3
We can also push down the `bid` selection.
![[w5n2plan3.png]]
This does not save any IOs, as we still need to scan `Reserves` the same number of times, for a cost $=250,500\text{ IOs}$. In general, pushing a selection into the inner loop of a nested loop join does not save IOs.
## Plan 4
We can swap which table is the outer loop of the join.
![[w5n2plan4.png]]
As 1% of the boats have a `bid` of 100, we get a cost $=1000+1000/100\cdot500=6000\text{ IOs}$.
## Plan 5
We could make a temporary table with the result of the sailor selection operation.
![[w5n2plan5.png]]
This requires $1000$ IOs to scan `Reserves`, $500$ IOs to scan `Sailors`, $250$ IOs to create the temporary table, and now for each page of the outer loop we only need to scan the temporary table, for a cost $=1000+500+250+10\cdot250=4250\text{ IOs}$.
## Plan 6
We can swap the loops again, this time using materialisation.
![[w5n2plan6.png]]
Here, the temporary table contains 10 pages, for a cost $=500+1000+10+250\cdot10=4010\text{ IOs}$.
## Plan 7
We can use [[W4N3 - Joins#Sort-merge join|sort-merge join]] instead of page nested loops.
![[w5n2plan7.png]]
It will cost $410+250=260\text{ IOs}$ to merge the sorted tables, and will require $1+\lceil\log_4(10/5)\rceil=2$ passes for `Reserves` with pass 0 $=10\text{ IOs}$ and pass 1 costing $2\cdot10=20\text{ IOs}$, and $1+\lceil\log_4(250/5)\rceil=4$ passes for `Sailors` with pass 0 $=250\text{ IOs}$ and passes 1, 2, and 3 costing $2\cdot250=500\text{ IOs}$. 

This makes a total cost of
$$
\begin{aligned}
\text{Total cost}&=\text{scan both}+\text{sort Reserves}+\text{sort Sailors}+\text{merge}\\
&=(1000+500)+(10+20)+(250+3\cdot500)+(260)\\
&=3540\text{ IOs}
\end{aligned}
$$
## Plan 8
Instead of using page nested loops, we can use block nested loops:
![[w5n2plan8.png]]
This will result in $\lceil250/3\rceil=84\text{ blocks}$, so we must iterate the temp table $84\cdot10$ times, for a total cost
$$
\begin{aligned}
\text{Total cost}&=\text{scan both}+\text{materialise}+\text{block nested loop join}\\
&=(500+1000)+(10)+(84\cdot10)\\
&=2350\text{ IOs}
\end{aligned}
$$
## Plan 9
We can cascade and pushdown the projection:
![[w5n2plan9.png]]
Now, we only have one iteration of the outer loop, so we only have to iterate `Sailors` once for a cost $=1000+1\cdot500=1500\text{ IOs}$. This is the best possible result without using indexes, as we will always have to scan both tables once at minimum.
## With indexes
If we have a clustered tree on `Reserves.bid` and an unclustered tree index on `Sailors.sid`, and assuming both fit in memory, we can produce the plan:
![[w5n2planWithIndexes.png]]
We now only need to access 10 pages of `Reserves`, and for each `Reserves` tuple (1000) we get the matching `Sailors` tuple (costs 1 IO), for a total cost $=10+1000\cdot1=1010\text{ IOs}$.