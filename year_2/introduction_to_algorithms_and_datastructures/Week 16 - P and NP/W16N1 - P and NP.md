# Decision problems
A **decision problem** is one in which we want to decide if something is true. Many optimisation problems can be re-cast into decision problems, such as "What is the edit distance between these two strings?" -> "Is the edit distance between these two strings less than 5?"

# Polynomial time
We have seen a range of algorithms in the course:
- [[W1N2 - Inefficient vs efficient algorithms#InsertSort|InsertSort]] - worst case run time $\Theta(n^2)$
- [[W1N2 - Inefficient vs efficient algorithms#MergeSort|MergeSort]] - worst case run time $\Theta(n\cdot\text{lg }n)$
- [[W9N2 - Breadth-first search|Breadth-first search]] - worst case run time $\Theta(m+n)$
- [[W12N2 - Edit distance|Edit distance]] - worst case run time $\Theta(m\cdot n)$
- [[W14N1 - Parsing context-free languages#The Cocke-Younger-Kasami Algorithm|CYK parsing]] - worst case run time $\Theta(n^3)$

All of these functions fit into the category of polynomial time.

A computational problem is in **polynomial time** if there exists a deterministic algorithm $A$ which correctly solves every instance of the problem and there is some fixed $r\in\mathbb{R}$ such that for every instance $J$ the algorithm runs in at most $O(|J|^r)$.

For example, InsertSort can sort any sortable list (i.e. a list that contains transitively comparable values) in $O(n^2)$. This means that $InsertSort\in P$, or InsertSort runs in polynomial time.

The complexity class $P$ includes all decision problems which can be solved by an algorithm that runs in polynomial time. Informally, we also often include non-decision problems (like edit distance or sorting) in $P$.

# Nondeterministic polynomial time
The complexity class $NP$ contains all decision problems where a potential solution can be validated in polynomial time. This means that $P\subset NP$, as by definition, if we can solve a problem in polynomial time we can validate that result in polynomial time.

# Reductions between problems
A problem $R$ can be reduced to the problem $Q$ if there is a polynomial-time function $f: \{0,1\}^*\rightarrow\{0,1\}^*$ (when defining problems in a two character language such as binary) such that for all instances $J$ of $R$:
$$
\begin{matrix}
	R(J)=1 & \Leftrightarrow & Q(f(J))=1
\end{matrix}
$$
This means that $R$ is no harder than $Q$ (in the context of polynomial-time computation), and that $Q$ is at least as hard as $R$.
This is written as $R\leq_PQ$.

- If $Q\in P$ then $R\in P$
- If $R$ is NP-complete then $Q$ is also NP-complete
($\leq_P$ is not like $\leq$ or $O()$, as we can ignore all polynomial factors)

# NP-completeness
A decision problem $Q$ is said to be **NP-complete** if $Q\in NP$ and for every problem $R\in NP$ then $R\leq_PQ$.
This is the equivalent to saying $Q$ is as hard as any problem in $NP$ can be.
Any NP problem can be reduced to any other NP-complete problem.

An example of an NP-complete problem is SAT.

# NP-hard
A decision problem $Q$ is **NP-hard** if $Q$ is harder than all problems in $NP$