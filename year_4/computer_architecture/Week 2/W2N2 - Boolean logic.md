There are two types of logic:
- **Combinatorial logic**: the outputs are solely dependent on the inputs
- **Sequential logic**: the outputs depend on present and past inputs, as the circuit has some form of memory
# Timing diagrams
![[w2n2timingDiagram.png]]
# Postulates and theorems of Boolean algebra
$$
\begin{matrix}
\text{Postulate 2} & x+0=x & x\cdot1=x\\
\text{Postulate 5} & x+x'=1 & x\cdot x'=0\\
\text{Theorem 1} & x+x=x & x\cdot x=x\\
\text{Theorem 2} & x+1=1 & x\cdot0=0\\
\text{Theorem 3, involution} & (x')'=x\\
\text{Postulate 3, commutative} & x+y=y+x & xy=yx\\
\text{Theorem 4, associative} & x+(y+z)=(x+y)+x & x(yz)=(xy)z\\
\text{Postulate 4, distributive} & x(y+z)=xy+xz & x+yz=(x+y)(x+z)\\
\text{Theorem 5, DeMorgan} & (x+y)'=x'y' & (xy)'=x'+y'\\
\text{Theorem 6, absorption} & x+xy=x & x(x+y)=x\\
\end{matrix}
$$
