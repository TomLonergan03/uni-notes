In general, [[W10N1 - Conjunctive queries|CQ]] [[W10N2 - Conjuctive query evaluation|evaluation]] is NP-hard. We can find, however, subclasses of CQs for which evaluation is tractable (in combined complexity).

Given a query:
$$
Q\text{ :- }R(x,y,z),R(z,u,v),R(v,w,x)
$$
We can produce a graph of it by making each variable a node and connecting edges between variables that appear in the same atom:
![[w11n2queryGraph.png]]

We can also produce a **hypergraph** (a generalisation of a graph where a **hyperedge** is any subset of nodes, vs in a graph where an edge is a subset of nodes with size 2) by creating a hyperedge for each atom:
![[w11n2queryHypergraph.png]]

A graph has a **treewidth**, which measures how close a graph is to a tree (a treewidth of 1 means it is a tree), and a graph has a **hypertree width**, which measures how close the hypergraph is to an acyclic hypergraph (which have a hypertree width of 1.

We will focus on CQs whose hypergraph have a hypertree width of 1, which are called **acyclic conjunctive queries**.
# Join trees
Given a hypergraph $H=(V,E)$, we can create a **join tree**, a labelled tree $T=(N,F,L)$ where $L:N\rightarrow E$ such that
1. For each hyperedge $e\in E$ there exists $n\in N$ such that $e=L(n)$ (so every hyperedge in the hypergraph should have a node in the tree)
2. For each node $u\in V$ of $H$, the set $\{n\in N|u\in L(n)\}$ induces a connected subtree of $T$ (so if we only focus on one node, we still get a connected subtree)

E.g.
![[w11n2joinTree.png]]
Here, we have a hypergraph and its corresponding join tree. If we select a specific node, here node $8$, we can see that the nodes in the tree containing that value still form a tree:
![[w11n2joinTreeConnectedSubtree.png]]

We can see that a cyclic hypergraph will not respect case 2, e.g:
![[w11n2cyclicJoinTree.png]]

We therefore define **acyclic hypergraphs** to be all hypergraphs which have join trees.

This means this hypergraph is cyclic:
![[w11n2cyclicHypergraph.png]]
while this one is no (as we have one hyperedge which is then connected to all other hyperedges):
![[w11n2acyclicHypergraph.png]]
# GYO-reduction
Given a conjunctive query $Q$, we want to determine whether $Q$ is acyclic. This is possible using a simple algorithm, **GYO-reduction**. This works by:
1. Eliminate all nodes that occur in at most one hyperedge
2. Eliminate all hyperedges that are empty or contained within another hyperedge
3. Repeat steps 1 and 2 until no more nodes and edges can be removed
4. If the resulting hypergraph is empty, it is acyclic. If it is not, then the hypergraph is cyclic

Checking acyclicity in this manner is in PTIME, or more specifically $O(|V|+|E|)$, and with more careful analysis (that we won't do here) it actually runs in $O(|Q|)$.
# Evaluating acyclic CQs
Acyclic CQs can be evaluated in PTIME using **Yannakaki's algorithm** using dynamic programming over the join tree. Given a database $D$ and an acyclic boolean CQ $Q$:
1. Compute the join tree $T$ of $H(Q)$
2. Assign to each node of $T$ the corresponding relation of $D$
3. Compute semi-joins in a bottom up traversal of $T$
4. Return YES if the resulting relation at the root of $T$ is non-empty, otherwise return NO

## Example
1. Build the join tree   ![[w11n2yannakakiStep1.png]]
2. Attach the corresponding relations![[w11n2yannakakiStep2.png]]
3. Compute the semi-joins from the bottom up![[w11n2yannakakiStep3.png]]
4. As the root relation is non-empty, this boolean CQ evaluates to true
