# Relational calculus
**Relational calculus** is a declarative way to describe a query using logical expressions. E.g. a query $\{x|Airport(x,London)\}$, which given the below tables will return all airport codes for airports in London.
![[w10n1relationalCalculusExample.png]]
# Conjunctive queries
**Conjunctive queries (CQ)** are the $\{\sigma,\pi,\bowtie\}$ subset of relational algebra, and is equivalent to both relational calculus without $\not,\forall,\vee,=$, and to simple `SELECT-FROM-WHERE` SQL queries with only `AND` and equality in the `WHERE` clause.
## Syntax
A conjunctive query is written in the form:
$$
Q(x):=\exists y(R_1(v_1)\wedge\dots\wedge R_m(v_m))
$$
where:
- $R_1,\dots,R_m$ are relation names
- $x,y,v_1,\dots,v_m$ are tuples of variables
- each variable mentioned in $v_i$ appears either in $x$ or in $y$
- the variables in $x$ are free and called **distinguished** or **output** variables

Conjunctive queries can be simplified to the form $Q(x):-R_1(v_1),\dots,R_m(v_m)$, known as the **body** of $Q$ and is the set of **atoms** which are conjuncted to form the query.
## Examples
Given the above tables, we can list all the airlines using the relational calculus $\{z|\exists x\exists y\ Flight(x,y,z)\}$, or with the equivalent conjunctive query $Q(z):\textendash Flight(x,y,z)$.

We can list the airport codes in London with the CQ $Q(x):-Airport(x,London)$.

We can list the airlines that fly directly from London to Glasgow using the relational calculus $\{z|\exists x\exists y\ Airport(x,London)\wedge Airport(y,Glasgow)\wedge Flight(x,y,z)\}$, and the equivalent CQ $Q(z):-Airport(x,London),Airport(y,Glasgow),Flight(x,y,z)$.
# Homomorphism
A **homomorphism** between a set of terms $S$ and a set of terms $T$ is a function $h:S\rightarrow T$, with $h$ being a set of mappings of the form $s\rightarrow t$, where $s\in S$ and $t\in T$.
A homomorphism from a set of atoms $A$ to a set of atoms $B$ is a substitution $h: terms(A)\rightarrow terms(B)$ such that:
- If $t$ is a constant value then $h(t)=t$
- If $R(t_1,\dots,t_k)\in A$ then $h(R(t_1,\dots,t_k))=R(h(t_1),\dots,h(t_k))\in B$ ($h$ respects the structure of $B$)

Given these sets:
![[w10n1homorphismSets.png]]

We can make a homomorphism $h_1=\{a\rightarrow a,b\rightarrow b,c\rightarrow c,d\rightarrow d,x\rightarrow a, y\rightarrow b\}$ which results in:
![[w10n1homomorphism1.png]]

And another homomorphism $h_2=\{a\rightarrow a,b\rightarrow b,c\rightarrow c,d\rightarrow d,x\rightarrow b, y\rightarrow c\}$ which results in:
![[w10n1homomorphism2.png]]

A **match** of a conjunctive query $Q(x_1,\dots,x_k):-body$ in a database $D$ is a homomorphism from the set of atoms $body$ to the set of atoms $D$. ^fc7176

The **answer** to that query is the set of $k$-tuples $Q(D):=\{(h(x_1),\dots,h(x_k))\ |\text{ h is a homomorphism between }Q\text{ and }D\}$.

Once we've found the answer, we can get the results of a CQ by applying each value of $h$ found to the free variables.