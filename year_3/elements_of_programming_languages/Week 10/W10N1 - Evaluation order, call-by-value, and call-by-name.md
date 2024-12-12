With [[W9N1 - Small-step semantics|small-step semantics]], we can divide the rules into 2 rough categories:
- Computational rules that actually do something, e.g
  $$
  \frac{}{v_1+v_1\rightarrowtail v_1+_\mathbb{N}v_2}
  $$
- Administrative rules that say how to evaluate within expressions, e.g.
  $$
  \begin{gathered}
	\frac{e_1\rightarrowtail e_1'}{e_1\oplus e_2\rightarrowtail e'_1\oplus e_2}&
	\frac{e_2\rightarrowtail e_2'}{v_1\oplus e_2\rightarrowtail v_1\oplus e'_2}
\end{gathered}
 $$

# Call-by-value
So far, function calls evaluate arguments to values before binding them to variables:
$$
\begin{gathered}
\frac{e_1\rightarrowtail e'_1}{e_1\;e_2\rightarrowtail e'_1\;e_2}&
\frac{e_2\rightarrowtail e'_2}{v_1\;e_2\rightarrowtail v_1\;e'_2}&
\frac{}{(\lambda x.e)\;v\rightarrowtail e[v/x]}
\end{gathered}
$$
This evaluation strategy is called *call-by-value* (or sometimes strict/eager evaluation), and is the default in most languages.
## Example
Consider $(\lambda x.x\times x)\;(1+(2\times3))$:
This is evaluated as:
$$
\begin{aligned}
(\lambda x.x\times x)\;(1+(2\times3))&\rightarrowtail(\lambda x.x\times x)\;(1+6)\\
&\rightarrowtail(\lambda x.x\times x)\;7\\
&\rightarrowtail7\times7\\
&\rightarrowtail49
\end{aligned}
$$
## Interpreting call-by-value
We evaluate subexpressions fully before substituting them for variables:
```scala
def eval(e: Expr): Value = e match {
	...
	case Let(x, e1, e2) = eval(subst(e2, eval(e1), x))
	...
	case Lambda(x, ty, e) = Lambda(x, ty, e)
	...
	case Apply(e1, e2) = eval(e1) match {
		case Lambda(x, _, e) = eval(subst(e, eval(e2), x))
	}
}
```

# Call-by-name
Call-by-value may evaluate expressions unnecessarily (leading to non-termination in the worst case), e.g.
$$
(\lambda x.42)\;loop\rightarrowtail(\lambda x.42)\;loop\rightarrowtail(\lambda x.42)\;loop\rightarrowtail(\lambda x.42)\;loop\rightarrowtail\cdots
$$
An alternative is to substitute expressions before evaluating, e.g.
$$
(\lambda x.42)\;loop\rightarrowtail42
$$
To do this, we remove the second administrative rule and generalise the computational rule:
$$
\begin{gathered}
\frac{e_1\rightarrowtail e'_1}{e_1\;e_2\rightarrowtail e'_1\;e_2}&\frac{}{(\lambda x.e_1)\;e_2\rightarrowtail e_1[e_2/x]}
\end{gathered}
$$
This evaluation strategy is called *call-by-name* (the "name" is the expression)
## Example
Consider $(\lambda x.x\times x)\;(1+(2\times3))$:
This is evaluated as:
$$
\begin{aligned}
(\lambda x.x\times x)\;(1+(2\times3))&\rightarrowtail(1+(2\times3))\times(1+(2\times3))\\
&\rightarrowtail(1+6)\times(1+(2\times3))\\
&\rightarrowtail7\times(1+(2\times3))\\
&\rightarrowtail7\times(1+6)\\
&\rightarrowtail7\times7\\
&\rightarrowtail7
\end{aligned}
$$
Note that the argument is computed twice!
## Interpreting call-by-name
We substitute expressions for variables *before* evaluating:
```scala
def eval(e: Expr): Value = e match {
	...
	case Let(x, e1, e2) = eval(subst(e2, e1, x))
	...
	case Lambda(x, ty, e) = Lambda(x, ty, e)
	...
	case Apply(e1, e2) = eval(e1) match {
		case Lambda(x, _, e) = eval(subst(e, e2, x))
	}
}
```
## In Scala
An argument can be passed by name by writing `=>` before the type. Such arguments are evaluated only when needed, but may be evaluated many times. e.g.
```scala
scala> def byname(x : => Int) = x + x
byName: (x: => Int)Int
scala> byName({ println("Hi there!"); 42})
Hi there!
Hi there!
res1: Int = 84
```
This can be useful, as sometimes we do want to evaluate an expression multiple times.
## Simulating call-by-name in a call-by-value language
We can do this by passing a lambda which yields the value, i.e. $\lambda().e:unit\rightarrow\\tau$, and when its value is needed, apply it to $()$.
