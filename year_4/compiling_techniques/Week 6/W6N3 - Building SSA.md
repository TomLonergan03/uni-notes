If we simply create a variable for each assignment in a program, we run into an issue: what happens with branching control flow.

E.g.
This program:
```
x = 0
if (a == 42)
	x = x + 1
else
	x = 3
y = x + 5
```
when converted to [[W6N1 - IR generation and SSA#Single static assignment|SSA]] is:
```
x1 = 0
if (a == 42)
	x2 = x1 + 1
else
	x3 = 3
y = x? + 5
```
but we don't know which variable x? should be.

At runtime, we would have to decide which of those two variables is used. This is done using a $\phi$ function.
![[w6n3phiNode.png]]
The $\phi$ function isn't a function in the programming sense, but instead a constraint on the compiler which tells it that at that point of the program both of those variables need to be in the same place i.e. register.
# $\phi$ function placement
The naive approach is to insert a $\phi$ function for every variable after every join. This is not great, as it will introduce many redundant $\phi$ functions, and produces the **maximal SSA form**.
![[w6n3naivePhiPlacement.png]]
# Basic blocks
A **basic block** is any sequence of instructions with no control flow, e.g.
```
a = 1 + 2
b = 2 * a
c = a - b
```
is a basic block.

```
a = 1 + 2
if b == 3 then
	b = b * 2
else
	b = 2
c = a + b
d = c + 1
```
has 4 basic blocks: `a = 1 + 2`, `b = b * 2`, `b = 2`, and `c = a + b; d = c + 1`.
# Dominators
A node $p$ **dominates** node $q$ iff every path from the entry node $b_0$ to $q$ also visits $p$. $Dom(q)$ is the set of nodes that dominate $q$.

E.g. for this graph
![[w6n3dominatorGraph.png]]

| $B$      | $B_0$ | $B_1$     | $B_2$     | $B_3$     | $B_4$         | $B_5$     | $B_6$ |
| -------- | ----- | --------- | --------- | --------- | ------------- | --------- | ----- |
| $Dom(B)$ | $B_0$ | $B_0,B_1$ | $B_0,B_1$ | $B_0,B_1$ | $B_0,B_1,B_4$ | $B_0,B_5$ | $B_0$ |
## Strict dominators
$p$ **strictly dominates** $q$ iff $p$ dominates $q$ and $p\ne q$.
## Dominance frontiers
$p$'s **dominance frontier** contains $q$ iff
- $p$ dominates a predecessor of $q$
- $p$ does not strictly dominate $q$

$DF(p)$ is the dominance frontier of $p$.

E.g. for the same graph as above

| $B$     | $B_0$         | $B_1$     | $B_2$ | $B_3$ | $B_4$     | $B_5$ | $B_6$         |
| ------- | ------------- | --------- | ----- | ----- | --------- | ----- | ------------- |
| $DF(B)$ | $\varnothing$ | $B_1,B_6$ | $B_4$ | $B_4$ | $B_1,B_6$ | $B_6$ | $\varnothing$ |
# Minimal SSA
We can produce a **minimal SSA form** by introducing a $\phi$ function in each node in $DF(B)$ for each variable that has a value assigned in $B$.

E.g. using the same graph as before:
![[w6n3minimalSsaBegin.png]]
$B_1$ and $B_6$ are in $DF(B_1)$, so variables assigned in $B_1$ must have $\phi$ functions in $B_1$ and $B_6$, and the same for each other node. We then get:
![[w6n3minimalSsaMiddle.png]]
and then rename each variable to get:
![[w6n3minimalSsaEnd.png]]
