>[!warning] This content is non-examinable

Conditionals, while-loops, exceptions, goto are all forms of control abstraction. Continuations are a highly general notion of control abstraction, which can be used to implement exceptions.

A continuation is a function representing "the rest of the computation", e.g:
```scala
def fact3[A](n: Int, k: Int => A): A = 
	if (n == 0) { k(1) }
	else { fact3(n - 1, {m => k(n * m)})}
```
This says: if `n` is 0, then pass 1 to `k`, otherwise, recursively call with parameters $n - 1$ and the continuation $\lambda r.k(n\times r)$. (`k` may be a function that simply prints its arg, or it may be the rest of a large program)

This function would evaluate as:
$$
\begin{aligned}
&\text{fact}3(3,\lambda x.x)\\
\rightarrowtail\;\;&\text{fact}3(2,\lambda r_1.(\lambda x.x)\;(3\times r_1))\\
\rightarrowtail\;\;&\text{fact}3(1,\lambda r_2.(r_1.(\lambda x.x)\;(3\times r_1))\;(2\times r_2))\\
\rightarrowtail\;\;&\text{fact}3(0,\lambda r_3.(\lambda r_2.(r_1.(\lambda x.x)\;(3\times r_1))\;(2\times r_2))\;1\times r_3))\\
\rightarrowtail\;\;&(\lambda r_3.(\lambda r_2.(r_1.(\lambda x.x)\;(3\times r_1))\;(2\times r_2))\;1\times r_3))\;1\\
\rightarrowtail\;\;&(\lambda r_2.(\lambda r_1.(\lambda x.x)\;(3\times r_1))\;(2\times r_2))\;(1\times1)\\
\rightarrowtail\;\;&(\lambda r_1.(\lambda x.x)\;(3\times r_1))\;(2\times1)\\
\rightarrowtail\;\;&(\lambda x.x) (3\times2)\\
\rightarrowtail\;\;&6
\end{aligned}
$$
# Interpreting $L_{Arith}$ using continuations
```scala
def eval[A](e: Expr, k: Value => A): A = e match {
	case Num(n) => k(NumV(n))

	case Plus(e1, e2) => 
		eval(e1, {case NumV(v1) =>
			eval(e2, {case NumV(v2) => k(NumV(v1 + v2))})})

	case Times(e1, e2) =>
		eval(e1, {case NumV(v1) =>
			eval(e2, {case NumV(v2) => k(NumV(v1 * v2))})})
}
```
Notice that when we evaluate `Plus(e1,...)` we use a different continuation to just continue with the rest of the `Plus` eval, instead of the existing `k`.
# Interpreting $L_{If}$ using continuations
```scala
def eval[A](e: Expr, k: Value => A): A = e match {
	...
	case Bool(n) => k(BoolV(n))

	case Eq(e1, e2) =>
		eval(e1, {v1 =>
			eval(e2, {v2 => k(BoolV(v1 == v2))})})

	case IfThenElse(e, e1, e2) =>
		eval(e, {case BoolV(v) =>
			if (v) { eval(e1, k) } else { eval(e2, k) }})
	...
}
```
# Interpreting $L_{Let}$ using continuations
```scala
def eval[A](e: Expr, k: Value => A): A = e match {
	...
	case Let(e1, x, e2) =>
		eval(e1, {v =>
			eval(subst(e2, v, x), k)}
	...
}
```
# Interpreting $L_{Rec}$ using continuations
```scala
def eval[A](e: Expr, k: Value => A): A = e match {
	...
	case Lambda(x, ty, e) => k(LambdaV(x, ty, e))
	case Rec(f, x, ty1, ty2, e) => k(RecV(f, x, ty1, ty2, e))

	case Apply(e1, e2) =>
		eval(e1, {v1 =>
			eval(e2, { v2 => v1 match {
			case LambdaV(x, ty, e) => eval(subst(e, v2, x), k)
			case RecV(f, x, ty1, ty2, e) =>
				eval(subst(subst(e, v2, x), v1, f), k)
			}})})
	...
}
```
# Interpreting $L_{Exn}$ using continuations
Here we must add a second continuation (`h`) to use for handling exceptions (and for the previous cases we can just propagate `h`):
```scala
def eval[A](e: Expr, h: Value => A, k: Value => A): A = e match {
	...
	case Raise(e0) => eval(e0, h, h)

	case Handle(e1, x, e2) =>
		eval(e1, {v => eval(subst(e2, v, x), h, k)}, k)
}
```
When we raise an exception, we replace `k` with `h` and use it instead. When handling, we create a new handler from `e2`
