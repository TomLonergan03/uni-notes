When [[W2N2 - Boolean logic|Boolean logic]] is implemented, we have these goals:
- minimise delay, area, and energy
- minimise number of gates
- minimise number of gate inputs (fan-in)
- minimise number of gates attached to each output (fan-out)
- minimise propagation delay
- minimise chip area
- minimise power consumption

Boolean logic must be minimised to its simplest form before synthesis to achieve these goals.
# Minterms and maxterms
Each row in a truth table defines a minterm and a maxterm.
**Minterms** indicate conditions where the function is true, and the function is therefore the OR of all minterms where the function is 1.
**Maxterms** indicate conditions where the function is false, and the function is therefore the AND of all maxterms where the function is 0.
![[w3n1mintermsAndMaxterms.png]]
Example:
![[w3n1minMaxTermExample.png]]
$$
\begin{matrix}
\text{Sum of minterms} & \text{Sum of maxterms}\\
F_1=\bar{x}\bar{y}z+x\bar{y}\bar{z}+xyz & F_1=(x+y+z)\times(x+\bar{y}+z)\times(x+\bar{y}+\bar{z})\times(\bar{x}+y+\bar{z})\times(\bar{x}+\bar{y}+z)\\
F_1=\bar{x}yz+x\bar{y}z+xy\bar{z}+xyz & F_2=(x+y+z)\times(x+y+\bar{z})\times(x+\bar{y}+z)\times(\bar{x}+y+z)
\end{matrix}
$$
# Logic minimisation
There are several approaches:
- Deductive methods: generally up to 3 variables, but is pretty error prone
- Karnaugh maps: up to 4 variables
- Quine-McClusky method: larger functions, outside the scope of this course
- Automated logic synthesis software: tends not to find the minimal solution, instead just finds one that fits within a set of constraints

## Deductive simplification example
$$
\begin{aligned}
F1&=\bar{x}yz+xyz+x\bar{y}\bar{z}\\
&=(x+\bar{x})yz+x\bar{y}\bar{z}\\
&=1yz+x\bar{y}\bar{z}\\
&=yz+x\bar{y}\bar{z}
\end{aligned}
$$
## Karnaugh maps
A Karnaugh map represents each of the minterms of a function, with each cell differing from its neighbours by one changed input.
![[w3n1karnaughMap.png]]
![[w3n1KarnaughFilled.png]]A Karnaugh map that shows a function $F=\bar{A}C\bar{D}+\bar{B}\bar{D}+\bar{B}\bar{C}$

A **prime implicant** is a term obtained by covering the maximum possible number of adjacent squares in the map. An **essential prime implicant** is a prime implicant which is the only implicant to cover a specific minterm.

Given a function $F(A,B,C,D)=\sum(0,2,3,5,7,8,9,19,11,13,15)$ (this means that minterms 0,2,3,etc. are cases where F returns true) we have:
![[w3n1karnaughExample.png]]
