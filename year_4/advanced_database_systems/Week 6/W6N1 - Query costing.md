For each [[W5N2 - Query optimisation#Plan space|plan]] we generate, we need to estimate its total cost. This requires estimating the cost of each operation in the plan tree based on input cardinalities, and the size of the result for each operation to determine the downstream input cardinalities. In system R, cost is a single number: $\text{cost}=\text{number of IOs}+\text{CPU factor}\cdot\text{number of tuples}$, with the second term being an estimate of the cost of tuple processing. In this course, we will use $\text{cost}=\text{number of IOs}$ as our approximation.
# Statistics and catalogues
**System catalogues** store internal statistics about tables, attributes, and indexes. They typically contain at least:
- **`NTuples`**: # of tuples in a table (cardinality)
- **`NPages`**: number of disk pages in a table or index
- **`Low/high`**: minimum and maximum value in a column
- **`NKeys`**: # of distinct values in a column
- **`Height`**: the height of an index
# Selectivity
The maximum output cardinality of an operator is the product of input cardinalities. Each term has a **selectivity (sel)**, which is the impact of the term in reducing result size. It is sometimes called the **reduction factor (RF)**. $\text{Selectivity}=|\text{output}|/|\text{input}|$, and will always be between 0 and 1. Note that a "highly selective" operator will have a low selectivity value.
## Estimating selectivity
The selectivity of equality predicate on unique keys are easy to estimate, as there is only one matching tuple.
For more complex predicates, we need heuristics.

If we have a query on `Students`, where attribute `age` has five distinct values:
```sql
SELECT * FROM Students WHERE age = 22
```
If we assume age is distributed evenly across its values, then$$
sel(A=constant)=1/NKeys(A)
$$In our case $sel(age=22)=1/5$.

For a range predicate:
```sql
SELECT * FROM Students WHERE age > 22
```
If we have a discrete (integer-valued) column:
$$
sel(A>a)=\frac{High(A)-a}{High(A)-Low(A)+1}
$$
and for continuous (floating-point) columns:
$$
sel(A>a)=\frac{High(A)-a}{High(A)-Low(A)}
$$
e.g. $sel(age>22)=(24-22)/(24-20+1)=2/5$

If we are comparing two attributes, then $sel(A=B)=1/\max\{NKeys(A), NKeys(B)\}$. We use $\max$ as if $A$ and $B$ are independent, and we have e.g. 2 distinct $A$ values $\{v_1,v_2\}$ and 10 distinct $B$ values $\{v_1,...,v_{10}\}$ then the probability of an $AB$ pair matching is:
$$
\begin{aligned}
P(A=B)&=\Sigma_iP(A=v_i,B=v_i)\\
&=\Sigma_iP(A=v_i)\cdot\Sigma_iP(B=v_i)\\
&=(1/2\cdot1/10)+(1/2\cdot1/10)+(0\cdot1/10)+...\\
&=1/10\\
&=1/\max\{2,10\}
\end{aligned}
$$
This is not perfect, as attributes may be correlated, but it makes a good approximation.

Negations have $sel(not P)=1-sel(P)$, e.g. $sel(age!=22)=1-1/5=4/5$.

A conjunction has $sel(P1\wedge P2)=sel(P1)\cdot sel(P2)$, and assumes predicates are independent.

A disjunction has $sel(P1\vee P2)=sel(P1)+sel(P2)-sel(P1\wedge P2)$, assuming predicates are independent.

Joins are just a selection over a big input $|R|\cdot|S|$, so produce $sel(p)\cdot|R|\cdot|S|$ total rows, so for an equi-join we produce $|R\bowtie S|\approx|R|\cdot|S|\cdot1/\max\{NKeys(R.A), NKeys(S.A)\}$ rows
# Missing statistics
DBMSes have hardcoded default selectivity values for various predicate types, which are used when it is not possible to estimate a selectivity value.
# Histograms
In practice, the distribution of data values is typically non-uniform. We can't keep the exact distribution for all attributes, so we use a histogram to approximate the distribution of the data. We do this by dividing the active domain of an attribute $A$ into adjacent intervals, and collect statistical parameters for each interval $(b_{i=1},b_i]$, e.g. the number of tuples $R$ with $b_{i-1}<r.A\leq b_i$ and the number of distinct $A$ values in interval $(b_{i-1},b_i]$. There are two common types of histograms:
## Equi-width histograms
All buckets have the same width $w$ (number of distinct values), i.e. $b_{i+1}=b_i+w$ for a fixed $w$.
If we have an actual value distribution of $A$ in relation $R$ containing 16 distinct values and 64 tuples:
![[w6n1valueDistribution.png]]

We then divide it into $B$ buckets, which means
$$
w=\frac{High(A,R)-Low(A,R)+1}{B}
$$
If we have $B=4$, we get $w=(16-1+1)/4=4$, and produce the following histogram:
![[w6n1equiwidthHistogram.png]]

Now we can use this information for our estimates, and assume a uniform distribution within the bucket.

For equality, if we are doing $Q\equiv\sigma_{k=5}(R)$, value 5 is in bucket $[5,8]$ with $19$ tuples, we get $|Q|=19/w=19/4\approx5$.

For ranges, if we do $Q\equiv\sigma_{A>7\text{ AND }A\leq16}(R)$, the interval covers buckets $[9,12]$ and $[13,16]$, and touches $[5,8]$, giving $|Q|=27+13+19/4\approx45$.
## Equi-depth histograms
All buckets contain the same number of tuples, but the width of buckets may vary. This is better able to adapt to data skew, as we don't lose information in buckets which contain large numbers of tuples.

We divide the distribution of $A$ into $B$ buckets, resulting in a depth $d=|R|/B$. We maintain depth $D$ and bucket boundaries $b_i$. E.g. for $B=4$, $D=64/4=16$, we make the histogram:
![[w6n1equidepthHistogram.png]]

Now for equality, if we do $Q\equiv\sigma_{k=5}(R)$, we get $|Q|=d/7=16/7\approx2$.

For range, if we do $Q\equiv\sigma_{A>7\text{ AND }A\leq16}(R)$, the range covers buckets $[8,9]$, $[9,10]$, and $[12,16]$, and touches $[1,7]$, for $|Q|=16+16+16+2/7\cdot16\approx53$.