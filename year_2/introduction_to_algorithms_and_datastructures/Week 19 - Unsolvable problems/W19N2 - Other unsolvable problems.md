# Russell's paradox
Define $R$ to be the set of all sets that don't contain themselves:
$$
R=\{S|S\notin S\}
$$
Does $R$ contain itself, i.e. is $R\in R$?

# Diophantine equations
A Diophantine equation is a multi-variable polynomial equation with integer coefficients for which we want integer solutions.
Examples of Diophantine equations include:
1. $x^2+y^2+z^2=42$
	- This is easy, as $|x|,|y|,|z|<7$ so we can perform a bounded search and find $5^2+4^2+1^2=42$
2. $x^3+y^3+z^3=42$
	- This is vastly more difficult, as cubes can be positive or negative
	  This was solved in 2019, with the equation $(-80538738812075974)^3+80435758145817515^3+12602123297335631^3=42$
3. $x^2y + 2yz^3 − 506zvw + w − v = 54321$

Given such an equation, does it have a solution?
This turns out to be unsolvable.

# Post's word problem
Given two finite sets $S,T$ of strings, decide if there is a string that can both be formed as a concatenation of strings in $S$ and as a concatenation of strings in $T$.
E.g.
$$
\begin{matrix}
S=\{a,ab,bba\}&T=\{baa,aa,bb\}
\end{matrix}
$$
In this case, the answer is yes as:
$$
bba.ab.bba.a=bbaabbba=bb.aa.bb.baa
$$
