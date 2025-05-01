# Query evaluation
## Combined complexity
We want to determine the complexity of evaluating a [[W10N1 - Conjunctive queries|conjunctive query]] over a database. We can't measure this in terms of output size, as simple queries may produce many results while complex ones produce perhaps only a single tuple, so we use **combined complexity**, which given a database $D$, a CQ $Q(x_1,\dots,x_k):-body$, and a tuple $(a_1,\dots,a_k)$ of values, determines if $(a_1,\dots,a_k)\in Q(D)$.
## Data complexity
**Data complexity** measures complexity in terms of the size of the database with a fixed query. This is meaningful in practice as a database is usually much larger than any query run on it, so we get a decision problem which given a database $D$ and a tuple $(a_1,\dots,a_k)$ determines if $(a_1,\dots,a_k)\in Q(D)$.

We now have our theorem for the complexity of query evaluation: CQ-evaluation's combined complexity is $NP$-complete, while its data complexity is in $P$.
# Aside on P and NP
A decision problem is in $P$ if a solution can be *found* in polynomial time. A decision problem is in $NP$ if a solution can be *verified* in polynomial time. $P\subseteq NP$, but it is an open problem whether $P\subset NP$ or $P=NP$ (this question is one of the Millennium Prize problems).

A decision problem is $NP$-complete iff it is in $NP$, and all other problems in $NP$ can be polynomial time reduced to it.
## 3-colourability problem
The 3COL problem asks given an undirected graph $G=(V,E)$, is there a function $c:V\rightarrow\{R,G,B\}$ such tha-t $\forall(v,u)\in E,\ c(v)\ne c(u)$? This is the process of colouring each vertex of the graph one of three colours, such that it does not share its colour with any of its neighbours.
![[w10n23colourability.png]]
3COL is $NP$-complete. It is in $NP$ as checking whether a graph is correctly coloured can be done by checking every pair $(u,v)\in E$, in $O(E+V)$ time. It is $NP$-complete as there exists a polynomial-time reduction from all other $NP$ problems (3-SAT is often used) but the proof of this is outside the scope of the course.
# Proof of complexity
We can show CQ-evaluation's combined complexity is in $NP$ using a guess and verify approach. 
## NP-membership
Given a fixed database $D$, a CQ $Q(x_1,\dots,x_k):-body$, and a tuple $(a_1,\dots,a_k)$ of values, we can guess a substitution $h:terms(body)\rightarrow terms(D)$ that is the identity on constants, and then verify that $h$ is a match of $Q$ in $D$ i.e. $h(body)\subseteq D$ and $(h(x_1),\dots,h(x_k))=(a_1,\dots,a_k)$.
## NP-hardness
We can reduce 3-COL to combined complexity using the lemma that a graph $G$ is 3-colourable iff $G$ can be mapped to $K_3$, i.e. $G$ is homomorphic to:
![[w10n2lemmaGK3.png]]
Therefore, we can create a database $D=\{E(x,y),E(y,z),E(z,x)\}$, and create a boolean CQ that represents the graph with each edge $(a,b)$ being an atom $E(a,b)$ in the query.
## Data complexity
Data complexity is in $P$ as we can simply test every substitution $h:terms(body)\rightarrow terms(D)$ that is the identity on constants, and check if $h(body)\subseteq D$ and $(h(x_1),\dots,h(x_k))=(a_1,\dots,a_k)$. As the number of terms in $body$ is fixed, it will take $O(\text{number of terms in query}\cdot\text{number of terms in }D)$ to check all entries.
# Static analysis
We can perform static analysis on CQs to answer questions about them without evaluating the query. There are 3 main analyses we can run:
## CQ-satisfiability
**CQ-satisfiability** asks that given a query $Q$, is there a database $D$ such that $Q(D)$ is non-empty? If there is no database, then we know the query is ill-formed, and evaluation is trivial - just return nothing.

We can prove all CQs $Q$ are satisfiable using a **canonical database**. We create the canonical database $D[Q]$ of a CQ $Q$ by replacing each variable in the body of $Q$ with a new value $c(x)=\underline{x}$, e.g. given $Q(x,y):\text{- }R(x,y),P(y,z,w),R(z,x)$ then $D[Q]=\{R(\underline{x},\underline{y}),P(\underline{y},\underline{z},\underline{w}),R(\underline{z},\underline{x})\}$. The mapping $c:\{\text{variables in body}\}\rightarrow\{\text{new variables}\}$ is a bijection, where $c(body)=D[Q]$ and $c^{-1}(D[Q])=body$.

As every query has a canonical database (we cannot have contradictions here due to the [[W10N1 - Conjunctive queries#Conjunctive queries|limitations of CQs]]), a CQ $Q$ is always satisfiable as $Q(D[Q])$ is trivially non-empty.
## CQ-equivalence
**CQ-equivalence** asks that given two queries $Q_1$ and $Q_2$, is $Q_1\equiv Q_2$ and equivalently, is it true that $\forall D,\ Q_1(D)=Q_2(D)$? This allows us to substitute one query for another that may be much simpler to run.
## CQ-containment
**CQ-containment** asks that given two queries $Q_1$ and $Q_2$, is $Q_1\subseteq Q_2$, and equivalently, is it true that $\forall D,\ Q_1(D)\subseteq Q_2(D)$?

If we can prove CQ-containment, we can prove CQ-equivalence, as $Q_1\equiv Q_2$ iff $Q_1\subseteq Q_2$ and $Q_2\subseteq Q_1$, so we only need to focus on CQ-containment.

We can define a query homomorphism from $Q_1(x_1,\dots,x_k)\text{:- }body_1$ to $Q_2(x_1,\dots,x_k)\text{:- }body_2$ as a substitution $h:terms(body_1)\rightarrow terms(body_2)$ such that:
- $h$ is a [[W10N1 - Conjunctive queries#Homomorphism|homomorphism]] from $body_1$ to $body_2$
- $(h(x_1),\dots,h(x_k))=(y_1,\dots,y_k)$

If $Q_1$ and $Q_2$ are conjunctive queries, then it holds that $Q_1\subseteq Q_2$ iff there exists a query homomorphism from $Q_2$ to $Q_1$.

We can prove this by first proving that if $Q_1\subseteq Q_2$ then there exists a query homomorphism from $Q_2$ to $Q_1$:
1. We know $(c(x_1),\dots,c(x_k))\in Q_1(D[Q_1])$
2. Since $Q_1\subseteq Q_2$, it must be that $(c(x_1),\dots,c(x_k))\in Q_2(D[Q_1])$
3. Therefore, there exists a homomorphism $h$ such that $h(body_2)\subseteq D[Q_1]$ and $h((y_1,\dots,y_k))=(c(x_1),\dots,c(x_k))$ (as there is a match for $Q_2$ in $D[Q_1]$, in other words there is a homomorphism from the atoms of $Q_2$ to the set of atoms in $D[Q_1]$ per the [[W10N1 - Conjunctive queries#^fc7176|definition of matches]])
4. By construction, $c^{-1}(c(body_1))=body_1$, and $c^{-1}((c(x_1),\dots,c(x_k)))=(x_1,\dots,x_k)$
5. Therefore, $c^{-1}\circ h$ is a query homomorphism from $Q_2$ to $Q_1$.

We now must prove that if there exists a query homomorphism from $Q_2$ to $Q_1$ then $Q_1\subseteq Q_2$:
1. Consider a database $D$ and a tuple $t$ such that $t\in Q_1(D)$. We want to show that $t\in Q_2(D)$
2. By the definition of a match, there exists a homomorphism $g$ such that $g(body_1)\subseteq D$ and $g((x_1,\dots,x_k))=t$
3. By the hypothesis, there exists a query homomorphism $h$ from $Q_2$ to $Q_1$
4. Therefore, $g(h(body_2))\subseteq D$ and $g(h((y_1,\dots,y_k)))=t$ which implies that $t\in Q_2(D)$

Therefore, $Q_1\subseteq Q_2$ iff there exists a query homomorphism from $Q_2$ to $Q_1$.
## Finding a query homomorphism
Given conjunctive queries $Q_1$ and $Q_2$, deciding whether there exists a query homomorphism from $Q_2$ to $Q_1$ is NP-complete.

We can prove that the problem is in NP by guessing a substitution, then check that it produces exactly $Q_1$ when applied to $Q_2$. This takes polynomial time as the time to check is $O(\text{number of atoms in }Q_1+\text{number of atoms in }Q_2)$.

The problem is $NP$-hard as we can reduce CQ-evaluation to a query homomorphism by converting our database into a query $Q_2$ containing all atoms in the database, and our value tuple into a query $Q_1$ containing exactly the values in the tuple, and determining whether there is a query homomorphism from $Q_2$ to $Q_1$ (maybe? not sure on this proof attempt).

As a result, deciding if a query homomorphism from $Q_2$ to $Q_1$ exists is NP-complete, and as a consequence of that, CQ equivalence and containment are also both NP-complete problems.