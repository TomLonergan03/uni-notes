A **Boolean variable** can take two values, 0 (FALSE) and 1 (TRUE).

A **Boolean function** $f(X)$ can only take the values 0 or 1.

A **Truth Table** is a representation of the output of a Boolean function with $n$ variables for all $2^n$ combinations of their values.

# Logic gates

| Operation              | OR              | AND              | NOT            |
| ---------------------- | --------------- | ---------------- | -------------- |
| Algebraic symbol       | $X + Y$         | $X\cdot Y$       | $\overline X$       |
| Circuit diagram symbol | ![[w1Or.png]]   | ![[w1And.png]]   | ![[w1Not.png]] |
| Truth table            | ![[w1OrTT.png]] | ![[w1AndTT.png]] | ![[w1NotTT.png]]               |

# Principles of Boolean algebra

| Law               | OR                                                       | AND                                                           |
| ----------------- | -------------------------------------------------------- | ------------------------------------------------------------- |
| Duality principle | $X+1=1$                                                  | $X\cdot 0=0$                                                  |
|                   | $X+0=X$                                                  | $X\cdot 1=1$                                                  |
| Idempotent law    | $X+X=X$                                                  | $Y+Y=Y$                                                       |
| Commutative law   | $X+Y=Y+X$                                                | $X\cdot Y=Y\cdot X$                                           |
| Associative law   | $(X+Y)+Z=X+(Y+Z)=X+Y+Z$                                  | $(X\cdot Y)\cdot Z=X\cdot (Y\cdot Z)=X\cdot Y\cdot Z$         |
| Distributivity    | $(X+Y)\cdot (X+Z)=X+ (Y\cdot Z)$                         | $(X\cdot Y)+(X\cdot Z)=X\cdot (Y+Z)$                          |
| Complementation   | $X+\overline X=1$                                             | $X\cdot \overline X=0$                                             |
| Absorption        | $X+X\cdot Y=X$                                           | $X\cdot (X+Y)=X$                                              |
|                   | $X+(\overline X\cdot Y)=X+Y$                                  | $X\cdot (\overline X+Y)=X\cdot Y$                                  |
| Logic adjacency   | $YX+Y\overline X=Y$                                           | $(Y+X)\cdot (Y+\overline X)=Y$                                     |
| Consensus         | $X\cdot Y+Y\cdot Z+Z\cdot \overline X=X\cdot Y+Z\cdot \overline X$ | $(X+Y)\cdot (Y+Z)\cdot (Z+\overline X)=(X+Y)\cdot (Z\cdot \overline X)$ |
| Double complement | $\overline{\overline X}=X$                                         |                                                               |
| De Morgan's       | $X\cdot Y=\overline{\overline X + \overline Y}$                    | $X+Y=\overline{\overline X \cdot \overline Y}$                    |

### Generalised De Morgan's
$X_1 \cdot X_2 \cdot ...\cdot X_N = \overline{\overline{X_1}+ \overline{X_2} +...+\overline{X_n}}$, dual: $X_1+X_2+...+X_N = \overline{\overline{X_1}\cdot \overline{X_2} \cdot ...\cdot \overline{X_n}}$

# Other logic gates
| Operation              | NAND                   | NOR                | XOR                                      |
| ---------------------- | ---------------------- | ------------------ | ---------------------------------------- |
| Algebraic symbol       | $\overline{X \cdot Y}$ | $\overline{X + Y}$ | $X\oplus Y=X\cdot\overline{Y}+\overline{X}\cdot Y$ |
| Circuit diagram symbol | ![[w1Nand.png]]        | ![[w1Nor.png]]     | ![[w1Xor.png]]                           |
| Truth table            | ![[w1NandTT.png]]      | ![[w1NorTT.png]]   | ![[w1XorTT.png]]                                         |

We can produce AND, OR and NOT (and by extension all Boolean operations) using only either NAND or NOR.

NOT can be made as: 
![[w1NorNot.png]] 
or
![[w1NandNot.png]]

OR can be made as:
![[w1NorOr.png]] 
or
![[w1NandOr.png]]
(by De Morgan's)

AND can be made as:
![[w1NorAnd.png]]
(by De Morgan's)
or
![[w1NandAnd.png]]

# Minterms and maxterms
A **minterm** (or fundamental product) is a Boolean expression with an AND term containing all direct or negated variables in the expression. 
E.g. $f(X_1,X_2,X_3)=X_1\cdot X_2\cdot  \overline{X_3}$ is a minterm

A **maxterm** (or fundamental sum) is a Boolean expression with an OR term containing all direct or negated variables in the expression.
E.g. $f(X_1,X_2,X_3)=X_1+\overline{X_2}+X_3$ is a maxterm