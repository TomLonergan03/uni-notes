A [[W18N1 - Computability#Register machines|register machine]] is said to halt if it eventually finishes computation, as opposed to looping forever.
As register machines and register values can be [[W18N1 - Computability#The universal machine|encoded]], we can define a register machine that is a halting tester.
A **halting tester** would take a register machine encoding $m$ and an initial memory state $n$ and output whether machine $m$ halts when run on initial memory state $m$.
![[w19n1HaltingTester.png]]

Theorem: There is no such register machine!
In other words, the halting problem is unsolvable.
Equivalently, the function $h(m,n)=(0\text{ if }m\text{ halts on input }n\text{, or }1\text{ otherwise})$ is not RM-computable.

Proof:
Supposing a halting tester exists, we could build the machine $P$, which consists of the halting tester, with the outputs set up so that if the halting tester states a program will halt, then $P$ does not halt, and equally if the passed in program does not halt, then $P$ halts.
If we are to pass $P$ the encoding of itself, then if $P$ halts then $P$ must not halt, and likewise if $P$ doesn't halt then $P$ does halt. This is a contradiction.