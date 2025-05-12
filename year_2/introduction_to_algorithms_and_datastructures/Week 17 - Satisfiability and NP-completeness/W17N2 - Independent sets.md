Given an undirected graph $G=(V,E)$ an **independent set (IS)** is a subset $I\subseteq V$ such that every pair $u,v\in I, (u,v)\notin E$, or a set of nodes ($I$) in $G$ such that no node in $I$ shares an edge with any other node in $I$.
The cardinality of an independent set ($|I|$) is the number of nodes within $I$.

We will consider the following decision problem:
Given an undirected graph $G=(V,E)$, and a natural number $k\in\mathbb{N}$, determine whether $G$ has an IS of size $\geq k$.
This decision problem is [[W16N1 - P and NP#NP-completeness|NP-complete]].

Example independent set:
![[w17n2IndependentSetGraph.png]]
This graph has a maximum $|I|=3$. $I$ is either $\{a,d,f\}$ or $\{b,d,e\}$.

# Proving that the independent set is NP-complete
We can prove this by [[W16N1 - P and NP#Reductions between problems|reducing]] [[W17N1 - Statisfiability#3-SAT|3-SAT]] ($R$) to the independent set. By definition of NP-complete this will show that the independent set is also NP-complete.

We can take a 3-SAT problem $\phi=C_1\land C_2\land...\land C_M$ where each $C_J=(l_{j,1}\lor l_{j,2}\lor l_{j,3})$ for three literals over $\{x_1,...,x_n\}$.

We can create a graph from this by creating edges between $l_{j,1},l_{j,2},l_{j,3}$, for all $j$, and creating an edge between every $l_j,=x_i$ and $l_j,=\overline{x_i}$. This means that either $x_i$ or $\overline{x_i}$ can be in the independent set/used to satisfy
the 3-SAT, but not both.

If we this graph has $|IS|=m$ then there is a satisfying assignment for $\phi$.

For example, given a 3-CNF formula $\phi=(x_2\lor\overline{x_4}\lor\overline{x_1})\land(x_4\lor\overline{x_2}\lor x_3)\land(x_1\lor x_2\lor\overline{x_4})$ we can construct the graph
![[w17n23CNFGraph.png]]
and find an independent set where $|IS|=3=m$
![[w17n23CNFIndependentSet.png]]
and therefore $\phi$ is satisfiable.

Our reduction contains $3\cdot m$ nodes, equal to the number of literals in the 3-CNF.
Number of edges is at most $3m+3\frac{m^2}{2}=O(m^2)$, (the number of connections within a group of 3 nodes plus the number of edges if all literals in each group is connected to every other group).

The construction of the graph can be done in polynomial time as building the triangles takes $O(m)$ and building the edges takes $O(m^2)$.

As a constructed graph with $|IS|\geq m\Leftrightarrow$ 3-CNF formula is satisfiable,
3-SAT $\leq_P$ Independent Set.