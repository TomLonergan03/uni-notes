# Minimal queries
A conjunctive query $Q_1$ is **minimal** if there is no CQ $Q_2$ such that:
1. $Q_1\equiv Q_2$
2. $Q_2$ has fewer atoms than $Q_1$

The task of **CQ minimisation** is, given a CQ $Q$, compute a minimal query that is equivalent to $Q$. This is desired as each atom in a CQ represents a [[W4N3 - Joins|join]], and fewer joins will almost always result in a faster executing query.
# Minimisation by deletion
Using [[W10N1 - Conjunctive queries#Homomorphism|homomorphism theory]], we can prove (but will not do so here) that if we have a CQ $Q_1(x_1,\dots,x_k)\text{ :- }body_1$, if $Q_1$ is equivalent to some CQ $Q_2(x_1,\dots,x_k)\text{ :- }body_2$ where $|body_2|<|body_1|$, then $Q_1$ is equivalent to some query $Q_3(x_1,\dots,x_k)\text{ :- }body_3$ such that $body_3\subseteq body_1$.

This means that we can minimise a CQ by simply deleting redundant atoms from its body, without having to search for any other rewriting of the query.

Therefore, we can perform minimisation by deleting any atom that does not contain an output variable from the body, then checking for [[W10N2 - Conjuctive query evaluation#CQ-equivalence|equivalence]] between the old and new queries. If they are equivalent, the new query is used as the base query, and repeat until none of the atoms in the body are able to be deleted.

E.g.
![[w11n1minimisationExample.png]]
Note that we can't make the query homomorphism $\{x\rightarrow a\}$ (to map $R(x,b)$ to $R(a,b)$) as $x$ is an output variable.
# Uniqueness of minimal queries
The order in which we remove atoms from the body of the input query does not matter, as if we have a CQ $Q$, and minimal CQs $Q_1$ and $Q_2$, then $Q_1\equiv Q$ and $Q_2\equiv Q$. This means that $Q_1$ and $Q_2$ must be isomorphic (i.e. they are the same, only differing in variable naming).

This means that given a CQ $Q$, the result of $Minimisation(Q)$ is unique (up to variable naming) and is called the **core** of $Q$.