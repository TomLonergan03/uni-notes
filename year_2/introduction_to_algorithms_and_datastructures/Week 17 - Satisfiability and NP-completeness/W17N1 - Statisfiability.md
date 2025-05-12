The canonical (and first proven) [[W16N1 - P and NP#NP-completeness|NP-complete]] problem is statisfiability.

Given a propositional logical formula $\phi$  over variables $\{x_1,...,x_n\}$ in conjunctive normal form (CNF) (in the format $\phi=C_1\land C_2\land C_3\land...\land C_n$ where each $C_i$ is some combination of $\{x_1,...,x_n\}$ ored together (e.g. $\phi_1=(x_1\lor x_2)\land(x_3\lor \overline x_4)$).

What assignments of logical variables makes $\phi$ true?
e.g. for $\phi_1$:
$x_1=0,x_2=0,x_3=0,x_4=0$ satisfies $\phi_1$.
$x_1=0,x_2=0,x_3=0,x_4=1$ does not satisfy $\phi_1$.

There are $2^n$ possible assignments to the $n$ logical variables of a CNF.

SAT: Given a CNF formula $\phi = C1 ∧ . . . ∧ Cm$ over variables $\{x1, . . . , xn\}$, determine whether there is some satisfying assignment for $\phi$.

# Cook-Levin Theorem
The Cook-Levin theorem proves that SAT is NP-complete, and therefore every NP problem $R$ reduces to SAT.

As there must exist a polynomial-time verifier for $R$, and we can consider all steps of verifying a solution as operations on binary data, we can encode these operations as Boolean operations on the binary data.
As the verifier is in polynomial time, the number of steps it takes is by definition polynomial. This means that the resultant CNF for the verifier is "polynomial-size".

# P=NP?
It is known that SAT, and other problems are NP-complete. It is, however, unknown whether P=NP or there are problems in P but not in NP. If P and NP are different, then at minimum, NP-complete problems lie outside of P.

# 3-CNF
A CNF formula $\phi$ is 3-CNF if each of its clauses $C_j$ is a disjunction of exactly 3 literals. E.g. $\phi=(x_1\lor x_2\lor\overline{x_3})\land(x_4\lor\overline{x_1}\lor x_3)$

# 3-SAT
Determine if there is an assignment of binary values that satisfies a 3-CNF problem $\phi$.