One approach: whenever we see $let x=e_1 in e_2$,
1. evaluate $e_1$ to $v_1$
2. replace $x$ with $v_1$ in $e_2$ and evaluate that

**Definition:** $e\Downarrow v$ for $L_{Let}$
$$
\frac{e_1\Downarrow v_1 \;\;\;e_2[v_1/x]\Downarrow v_2}{\text{let }x=e_1\text{ in }e_2\Downarrow v_2}
$$
With this definition we always substitute a value for a variable, so we don't need a rule for "evaluating" a variable. This approach is called *eager* or *strict* evaluation.

# Interpreter changes
```scala
type Variable = String
...
case class Var(x: Variable) extends Expr
case class Let(x: Variable, e1: Expr, e2: Expr) extends Expr
...
def eval(e: Expr): Value = e match {
	...
	case Let(x, e1, e2) => {
		val v = eval(e1);
		val e2vx = subst(e2, v, x);
		eval(e2vx)
	}
}
```
We don't need a case for `Var(x)` as all variables must be bound at some point.
# Types and variables
We have to keep track of the types of variables (probably with a table or map). This means that for typechecking, whenever we see a variable $x$ we look up its type in the map. When we see a $\text{let }x=e_1\text{ in }e_2$, we find out the type of $e_1$, which is $\tau_1$. We can then add that to the map, and check $e_2$ using the new map. It is important that the information about the type of $x$ does not persist beyond its scope in $e_2$.

## Type environments
We write $\Gamma$ to denote a **type environment**, or a finite map from variable names to types, often written as follows:
$$
\Gamma::=x_1:\tau_1,...,x_n:\tau_n
$$
Also, we write $\Gamma(x)$ to mean the type of $x$ according to $\Gamma$, and $\Gamma,x:\tau$ to indicate extending $\Gamma$ with the mapping $x$ to $\tau$.

## Well-formedness in a context
We write $\Gamma\vdash e:\tau$ to indicate that $e$ is well-formed at type $\tau$ (or just "has type $\tau$") in context $\Gamma$.

We also need to generalise the $L_{If}$ rules to allow contexts:
$$
\begin{matrix}
\frac{}{\Gamma\vdash n:\text{ int}} &
\frac{\Gamma\vdash e_1:\tau_1\;\;\;\;\Gamma\vdash e_2:\tau_2\;\;\;\;\oplus:\tau_1\times\tau_2\rightarrow\tau}{\Gamma\vdash e_1\oplus e_2:\tau}\\
\frac{}{\Gamma\vdash b:\text{ bool}}&
\frac{\Gamma\vdash e:\text{ bool}\;\;\;\;\Gamma\vdash e_1:\tau\;\;\;\;\Gamma\vdash e_2:\tau}{\Gamma\vdash\text{ if }e\text{ then }e_2\text{ else }e_2:\tau}
\end{matrix}
$$
