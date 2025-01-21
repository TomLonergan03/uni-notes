There are two relational query languages:
1. **Relational calculus**: serves as the basis for SQL and describes the result of a computation based on first order logic. E.g. $\left\{S|S\in Student\ \exists E\in Enrolled\ (S.sid=E.sid\wedge E.cid=\text{'INF-11199'}\right\}$
2. **Relational algebra**: an algebra on sets that is an operational description of transformations
# Codd's theorem
**Codd's theorem** states that any relational calculus can be expressed as a relational algebra, and vice versa.
# Relational algebra
**Relational algebra** is an algebra of operators on relation instances, e.g. $\pi_{S.name}(\sigma_{E.cid='INF-11199'}(S\bowtie_{S.sid=E.sid}E))$. It is closed (so the result of any operator is also a relation instance, enabling composition), and typed (so the input schema determines the output schema, allowing for correctness of queries to be determined statically).

Some key relation operators include:
- **Selection** ($\sigma_{predicate}(R)$) selects a subset of rows that satisfy the selection predicate, with the output schema being the same as the input.
- **Projection** ($\pi_{A_1,A_2,...,A_n}(R)$) selects a subset of columns, with the output schema determined by the schema of the attribute list.
- **Union** ($R\cup S$) joins two relations where both have the same number of fields, and the fields in each have the same types in the same order.
- **Set difference** ($R-S$) returns all tuples in $R$ that are not in $S$, where both relations have the same field types.
- **Cross product** ($R\times S$) will produce every row $R$ paired with each row of $S$, resulting in $|R|\cdot|S|$ rows in the result. The output schema will be the concatenation of the two input schemas. 
- **Renaming** ($\rho(OutputName(1\rightarrow name1, 2\rightarrow name2), R)$) will produce a relation called $OutputName$ with attribute 1 called $name1$ and attribute 2 called $name2$, with the same contents as $R$. 
- **Intersection** ($R\cap S$) returns the tuples in both $R$ and $S$, requiring both relations to be compatible. It is equivalent to $R-(R-S)$.
- **Join** ($\bowtie$): there is a hierarchy of common kinds of joins:
	- **Theta joins** ($R\bowtie_\theta S$) joins $R$ and $S$ on a logical expression $\theta$. It is the equivalent of $\sigma_\theta(R\times S)$.
	- **Equi-join** is a theta join with theta being a conjunction of equalities.
	- **Natural join** ($R\bowtie S$) is an equi-join on all matching column names. It is the equivalent of $\pi_\text{unique fields}(\sigma_\text{matching fields are equal}(R\times S))$.

There are other operators that are sometimes supported:
- **Aggregation** ($\gamma_\text{attributes, aggregation function, <selection criteria>}(R)$) produces aggregated results of a function over the relation.
- **Duplicate elimination** ($\delta(R)$) eliminates duplicates in a relation. It is only relevant under a multiset interpretation of relational algebra.
- **Assignment** ($R\leftarrow S$) assigns a value to a name, such as one produced through aggregation
- **Sorting** ($\tau_\theta(R)$) sorts a relation by a comparison operator $\theta$