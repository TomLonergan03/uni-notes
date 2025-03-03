# Reaching analysis
A definition of a variable $v$ at a point in the program $d$  **reaches** a point $u$ if there exists a control-flow path $p$ from $d$ to $u$ such that no definition of $v$ appears on that path.
![[w7n1variableReach.png]]
($a$ defined at $s_3$ reaches $s_8$ via $s_7$)
## Local reaching analysis
A **local analysis** works on a single [[W6N3 - Building SSA#Basic blocks|basic block]].

**Local reaching analysis** works by:
- Maintain a set of current reaching definitions
- Add subscripts to all variable definitions
- Go through all statements from start to end
- If an assignment statement $x_i$ is found
	- Remove all other $x_j$ from the set
	- Add $x_i$ to the set

![[w7n1localReachingAnalysis.png]]
## Global reaching analysis
We can perform this analysis globally by redefining a program point so that each statement has an $In$ program point for entering the statement, and an $Out$ program point for leaving the statement. We then taking the union of reaching variables at each $In$ point, and iterating until no more changes occur.

E.g. given this graph:
![[w7n1globalReaching1.png]]
We iterate once and get:
![[w7n1globalReaching2.png]]
Notice $In(s_5)=Out(s_3)\cup Out(s_4)$, and that $s_4$ is not yet complete.
As we made changes in iteration 1, we do another and get:
![[w7n1globalReaching3.png]]
Now $s_4,s_5,s_6$ are complete. If we iterate again there are no more changes, so we are done.
### Formally
- For each statement $s$ compute $Out(s)$ from $In(s)$. If $s$ is an assignment to $x$, delete all definitions of $x$ and add new definitions: $Out(s:x_i:=...)=(In(s)-\{x_j;\forall j\})\cup\{x_i\}$.
- Multiple incoming edges are merged to compute $In(s)$: $In(s)=\bigcup_{\forall p\in Pred(s)}Out(p)$.
- All nodes start with an empty set of reaching variables $Init(s)=\varnothing$.
# General dataflow analysis
We can generalise this concept for any other desired dataflow analysis. An analysis must have:
- a **direction**: either from the program start to end, or end to start.
- a **transfer function** that computes the effect of a statement, e.g. $Out(s)=(In(s)-Kill(s))\cup Gen(s)$
- a **meet operator** that merges values from multiple incoming edges, e.g. $In(s)=\bigcup_{\forall p\in Pred(x)}Out(p)$.
- the **value set**: information being passed around.
- **initial values**: the most conservative values for all nodes, and often a special case for the start node.

And for a general iterative algorithm we have:
```
for each node, start_node do
	Initialise start_node

while values changing do
	for each node do
		Apply meet function // compute In(s)
		Apply transfer function // compute Out(s)
```
As long as $Out(s)$ is strictly growing, or strictly shrinking, termination is guaranteed, but in the general case this is NP-hard.
## Performance
The round robin algorithm is slow and may require many passes through nodes, with the direction of traversal making a big difference. This can be sped up by considering basic blocks instead of individual nodes, and by tracking which nodes inputs change, as only they need reprocessed.
# Liveness analysis
A variable is **live** at a program point if it's current value may be read during the remaining execution of the program, otherwise it is **dead**.

Formally, a variable $v$ is live before a CFG node $s$ if:
- $v\in use_{var}(s)$, or
- $\exists$ a direct path from $s$ to a node that uses $v$, and that path does not go through a node that defines $v$.

As a dataflow analysis, we have:
- **direction**: backwards
- **transfer function**: $live(n)=(candidates(n)-def_{var}(n))\cup use_{var}(n)$.
- **meet operator**: $candidates(n)=\bigcup_{\forall s\in Succ(n)}live(s)$.
- **value set**: a set of variables and a set of candidates
- **initial values**: $\varnothing$.

E.g.
Given the program:
![[w7n1liveness1.png]]
Our first iteration gives us:

| Node $n$ | $candidate(n)$ | $live(n)$     |
| -------- | -------------- | ------------- |
| 13       | $\varnothing$  | $\varnothing$ |
| 12       | $\varnothing$  | $\{x\}$       |
| 11       | $\varnothing$  | $\{z\}$       |
| 10       | $\{z\}$        | $\{x,z\}$     |
| 9        | $\{x,z\}$      | $\{x,z\}$     |
| 8        | $\{x,z\}$      | $\{x\}$       |
| 7        | $\{x\}$        | $\{x,y\}$     |
| 6        | $\{x,y\}$      | $\{x,y\}$     |
| 5        | $\{x,y\}$      | $\{x\}$       |
| 4        | $\{x\}$        | $\{x\}$       |
| 3        | $\{x\}$        | $\varnothing$ |
| 2        | $\varnothing$  | $\varnothing$ |
| 1        | $\varnothing$  | $\varnothing$ |
Then iterating again, we get one change: $s_{11}$ gets $x$ as a candidate from $s_{4}$, which updates $live(11)$:

| Node $n$ | $candidate(n)$ | $live(n)$     |
| -------- | -------------- | ------------- |
| 13       | $\varnothing$  | $\varnothing$ |
| 12       | $\varnothing$  | $\{x\}$       |
| 11       | ${x}$          | $\{x,z\}$     |
| 10       | $\{z\}$        | $\{x,z\}$     |
| 9        | $\{x,z\}$      | $\{x,z\}$     |
| 8        | $\{x,z\}$      | $\{x\}$       |
| 7        | $\{x\}$        | $\{x,y\}$     |
| 6        | $\{x,y\}$      | $\{x,y\}$     |
| 5        | $\{x,y\}$      | $\{x\}$       |
| 4        | $\{x\}$        | $\{x\}$       |
| 3        | $\{x\}$        | $\varnothing$ |
| 2        | $\varnothing$  | $\varnothing$ |
| 1        | $\varnothing$  | $\varnothing$ |
On the third iteration, nothing changes, so we are done. As $y$ and $z$ are never live at the same time, this allows us to merge them into one variable, e.g. they can both be allocated to the same register.
# Limitations
Dataflow analysis has some limitations:
- Static analysis may be very conservative relative to the actual dynamic control flow.
- Pointers introduce problems, as if `*x = 10` we may not be able to know whether `x` points to another variable `y`, and similarly for arrays with non-constant indexes
- Reasoning across function calls quickly blows up into massive graphs.