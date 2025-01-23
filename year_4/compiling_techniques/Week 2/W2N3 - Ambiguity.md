A grammar is **ambiguous** if there is more than one derivation for a single sentence. This is a problem for building an internal representation.
E.g. with the grammar:
$$
\begin{aligned}
Expr&::=Expr\ Op\ Expr\ |\ num\ |\ id\\
Op\ &::=+\ |\ *
\end{aligned}
$$
We have multiple leftmost derivations for $x+2*y$:
$$
\begin{aligned}
&Expr\\
&Expr\ Op\ Expr\\
&id(x)\ Op\ Expr\\
&id(x)\ +\ Expr\\
&id(x)\ +\ Expr\ Op\ Expr\\
&id(x)\ +\ num(2)\ Op\ Expr\\
&id(x)\ +\ num(2)\ *\ Expr\\
&id(x)\ +\ num(2)\ *\ id(y)\\\\
&\text{Parsed as $x+(2*y)$}
\end{aligned}
$$
or
$$
\begin{aligned}
&Expr\\
&Expr\ Op\ Expr\\
&Expr\ Op\ Expr\ Op\ Expr\\
&id(x)\ Op\ Expr\ Op\ Expr\\
&id(x)\ +\ Expr\ Op\ Expr\\
&id(x)\ +\ num(2)\ Op\ Expr\\
&id(x)\ +\ num(2)\ *\ Expr\\
&id(x)\ +\ num(2)\ *\ id(y)\\\\
&\text{Parsed as $(x+2)*y$}
\end{aligned}
$$
E.g. with the grammar:
$$
\begin{aligned}
Stmt::=\ &if\ Expr\ then\ Stmt\\
|\ &if\ Expr\ then\ Stmt\ else\ Stmt\\
|\ &OtherStatement
\end{aligned}
$$
We have multiple leftmost derivations for $if\ E1\ then\ if\ E2\ then\ S1\ else\ S2$:
```
if E1 then
	if E2 then
		S1
else
	S2
```
or
```
if E1 then
	if E2 then
		S1
	else
		S2
```
There are also deeper ambiguities, e.g. in Algol `a=f(17)` `f` can either be a function or an array. 
# Resolving ambiguity
We can remove context-free grammar ambiguity by rewriting the grammar, e.g. in the case of $if-then-else$:
$$
\begin{aligned}
Stmt::=\ &if\ Expr\ then\ Stmt\\
|\ &if\ Expr\ then\ WithElse\ else\ Stmt\\
|\ &OtherStmt\\
\\
WithElse::=\ &if\ Expr\ then\ WithElse\ else\ WithElse\\
|\ &OtherStmt
\end{aligned}
$$
This pushes elses to the bottom of the tree (so always parses as the second derivation given above)

Context sensitive ambiguity, e.g. function vs array, can be left to be resolved by the semantic analyser.