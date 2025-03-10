![[w7n2registerAllocation.png]]
**Register allocation** is a backend pass that assigns [[W6N1 - IR generation and SSA#Single static assignment|SSA]] variables to specific registers in the CPU.
# Definitions
- **Allocation** is deciding which values to keep in registers.
- **Assignment** is choosing the specific registers for values.
- **Liveness**: a value is [[W7N1 - Dataflow analysis#Liveness analysis|live]] between its definition and its last use.
- **Interference**: two values that are live at the same time **interfere** and cannot be placed in the same register.
- **Live range**: the set of statements in which a value is live. It may be conservatively overestimated, e.g. live for the whole program.
- **Spilling**: saving a value from a register to memory, freeing the register for another value. Requires $F$ registers to be reserve to keep track of where spilled values are, usually on the stack.
- **Clean and dirty values**: a spilled value is **clean** if it has not changed since its last spill, otherwise it is **dirty**. A clean value can be spilled without a new store instruction.
# Local register allocation
Allocating registers in a single basic block is local allocation. If we have a maximum of $l$ values live at any point in the block, and $k$ registers, then if $l\leq k$ then allocation is straightforward and there is no need to reserve $F$ registers for spilling. If $l>k$ then some values will be spilled to memory and we need to reserve $F$ registers for spilling. 
## Top down allocation
If we have more than $k$ values, then rank values by occurrence, and allocate the first $k-F$ values to registers and spill the other values.

E.g. with $k=2,F=0$:
![[w7n2localTopDown.png]]
## Bottom up
Start with an empty register set, and load values on demand. When no register is available, free one using a heuristic. In this case we will spill the value whose next use is farthest in the future (easy to determine for local allocation, possibly impossible for global allocation), preferring clean values to dirty ones.

E.g. with $k=2,F=0$:
![[w7n2localBottomUp.png]]
# Global register allocation
**Global allocation** allows for reuse of values across multiple blocks. Most modern global allocators use a graph-colouring paradigm, first building an interference graph with edges between interfering values, then finding a $k$-colouring for the graph, or changing the code to a nearby problem that can be $k$ coloured. It is an NP-complete problem under nearly all assumptions (though local allocation is also NP-complete with dirty vs clean).
## Graph colouring
A graph $G$ is said to be $k$ colourable if the nodes can be labelled with integers $1..k$ such that no edge in $H$ connects two nodes with the same label.
![[w7n2graphColouring.png]]
## Interference graph
An interference graph $G=(N,E)$ where nodes in $G$ represent values, while edges represent individual interferences, i.e. $\forall x,y\in N,x\rightarrow y\in E$ iff $x$ and $y$ interfere. A $k$-colouring of $G$ can be mapped into an allocation to $k$ registers.
## Colouring the register graph
The degree of a node $n\degree$ is a loose upper bound on colourability, with any node $n$ such that $n\degree<k$ is always trivially $k$-colourable, as trivially colourable nodes cannot adversely affect the colourability of neighbours, so can be removed from the graph. This reduces the degree of its neighbours, which may then be trivially colourable. If we are left with any nodes such that $n\degree\geq k$ spill one, and try again.
## Chaitin's algorithm
1. While $\exists n\in N,n\degree<k$, then $n$ can be removed and pushed to a stack.
2. If $G$ is still non-empty then pick a $n\in N$ using a heuristic and add it to the spill list. It is then removed from $G$, then go back to step 1.
3. If the spill list is not empty, insert the spill code, then rebuild the interference graph and try to allocate again.
4. Otherwise, pop vertices off the stack and colour them in the lowest use colour not used by a neighbour.

E.g.
If we start with this graph:
![[w7n2chaitin1.png]]
we can select $a$ and push it to the stack:
![[w7n2chaitin2.png]]
then push $b$, as it now has 2 neighbours, then $c,d,e$ similarly:
![[w7n2chaitin3.png]]
Now, we can pop $e$ and choose a colour for it, red in this case:
![[w7n2chaitin4.png]]
then pop $d$:
![[w7n2chaitin5.png]]
and likewise for $c,b,a$ to get a final graph:
![[w7n2chaitin6.png]]
### Optimistic colouring
Chaitin's algorithm may spill nodes unnecessarily, and the Chaitin-Briggs algorithm attempts to resolve it by also pushing nodes with degree $\geq k$ to the stack after all trivial nodes, and once popping values if some nodes are not colourable, then it will pick one of the uncoloured vertexes to spill, then restart.

E.g. given this graph with $k=2$, Chaitin's algorithm will immediately spill a node, even though the graph is two colourable.
![[w7n2chaitinBriggs.png]]
## Selecting spill candidates
When selecting a candidate for spilling, there are a number of things to consider:
- High degree nodes are more likely to help colouring.
- Higher usage values (e.g. inner loop values) are more expensive to spill, as they will be loaded and stored more frequently.
- A definition immediately followed by a use has an infinite spill cost as spilling does not decrease live range.
## Live range splitting
A value may interfere with multiple other live ranges, and splitting a value into multiple smaller live ranges may produce a colourable graph.
## Coalescing
If two ranges don't interfere and are connected by a copy, they can be coalesced into one, reducing the degree of nodes that interfered with both. I.e. if $x=y$ and $x\rightarrow y\in G$ then combine $LR_x$ and $LR_y$. As this reduces degree, it is often applied before colouring takes place.

Coalescing can make a graph harder to colour, as typically $xy\degree>\max(x\degree, y\degree)$. If $\max(x\degree,y\degree)<k<xy\degree$ then $xy$ might spill while $x$ and $y$ would not spill.
![[w7n2coalescingDegree.png]]
### Conservative coalescing
We only need to coalesce $x$ and $y$ if $xy$ has $<k$ neighbours of degree $>k$, as only neighbours of significant degree can force $xy$ to spill. These nodes are always safe to coalesce, as it cannon introduce a node of non-trivial degree and cannot introduce a new spill.
## Other approaches
Top-down colouring uses high level priorities to decide on colouring, e.g.
- Control flow structure can guide allocation
- Enumerating all combinatorial options can yield improvements, but is very expensive
- If it is simple to recreate a value it can be rematerialised later instead of spilling it.
- Linear scans are a fast and weak way to determine allocation, and can be useful in JITs.
