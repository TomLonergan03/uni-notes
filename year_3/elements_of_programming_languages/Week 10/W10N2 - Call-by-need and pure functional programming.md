# Call-by-value vs call-by-name
- [[W10N1 - Evaluation order, call-by-value, and call-by-name#Call-by-value|Call-by-value]] evaluates every expression once, whether or not its value is needed. This means that performance tends to be more predictable, and side-effects happen predictably
- [[W10N1 - Evaluation order, call-by-value, and call-by-name#Call-by-name|Call-by-name]] only evaluates an expression if its value is needed, meaning that it can be faster (or even avoid infinite loops) if a value is not needed, but may evaluate multiple times if the value is needed in many places. Reasoning about performance requires understanding when expressions are needed, and side-effects may happen many times or not at all
# Call-by-need
We can unify some of the benefits of both approaches by evaluating each expression the first time it is needed, but then we save the result. This means that if an expression's value is never needed, it never gets evaluated, whereas if it is needed many times, it's sill only evaluated once. This is called *call-by-need* or *lazy* evaluation.
## Laziness in Scala
Scala provides a `lazy` keyword for `val`s, which means that variable is not evaluated until it is needed.
```scala
scala> lazy val x = {println("Hello"); 42}
x: Int = <lazy>
scala> x + x
Hello
res0: Int = 84
```
(Note that "Hello" is printed once, but `42+42` is still calculated)
## Emulating laziness in Scala
```scala
class Lazy[A](a: => A) {
	private var r: Either[A, () => A] = Right{() => a}
	def force = r match {
		case Left(v) => v
		case Right(f) => {
			val a = f()
			r = Left(a)
			a
		}
	}
}
```
## Rules for call-by-need
$$
\begin{gathered}
\frac{}{\sigma,(\lambda x.e_1)\;e_2\rightarrow\sigma[l:=e_2],e_1[l/x]}\\
\frac{}{\sigma,\text{let }x=e_1\text{ in }e_2\rightarrowtail\sigma[l:=e_1],e_2[l/x]}\\
\frac{}{\sigma[l:=v],l\rightarrowtail\sigma[l:=v],v}\\
\frac{\sigma,e\rightarrowtail\sigma',e'}{\sigma[l:=e],l\rightarrowtail\sigma'[l:=e'],l}
\end{gathered}
$$
When we reduce a function application or use $\text{let}$, we add the expression to the memo table and replace it with a label. When we encounter the label, we look it up and evaluate the expression in the memo table (if it has been previously evaluated then it will simply be a value).

As with [[W9N4 - References and Arrays|L_Ref]] we also need to adjust all of the rules to handle $\sigma$:
$$
\begin{gathered}
\frac{\sigma,e_1\rightarrowtail\sigma',e'_1}{\sigma,e_1\oplus e_2\rightarrow\sigma',e'_1\oplus e_2}\\
\frac{\sigma,e_2\rightarrowtail\sigma',e'_2}{\sigma,v_1\oplus e_2\rightarrow\sigma',v_1\oplus e'_2}\\
\frac{}{\sigma,v_1+v_2\rightarrowtail\sigma,v_1+_\mathbb{R}v_2}\\
\frac{}{\sigma,v_1\times v_2\rightarrowtail\sigma,v_1\times_\mathbb{R}v_2}\\
\vdots
\end{gathered}
$$
## Example
Consider $(\lambda x.x\times x)\;(1+(2\times3))$:
This is evaluated as:
$$
\begin{aligned}[]
[],(\lambda x.x\times x)\;(1+(2\times3))&\rightarrowtail[l=1+(2\times3)],l\times l\\
&\rightarrowtail[l=1+6],l\times l\\
&\rightarrowtail[l=7],l\times l\\
&\rightarrowtail[l=7],7\times l\\
&\rightarrowtail[l=7],7\times7\\
&\rightarrowtail[l=7],49
\end{aligned}
$$
Here, the argument is computed only once, and only at the point where it is needed
# Pure functional programming
Call-by-name/call-by-need interact badly with side effects, but on the other hand, they support very strong equational reasoning abut programs. Haskell (and some other languages) are pure, which is to say they adopt lazy evaluation and forbid any side effects.
This has strengths and weaknesses:
- Strengths:
	- Easier to optimise and parallelise as side-effects are forbidden
	- Can be faster
- Weaknesses:
	- Memoisation has overhead, and performance is less predictable
	- Side effects are actually very important for programs, e.g. I/O, so there is a lot of thought required to get them working
## IO in Haskell
The solution for dealing with desired side-effects such as I/O, Haskell uses a type constructor (`IO a`) to "encapsulate" side-effecting computations, e.g.
```haskell
do {x <- readLn::IO Int; print x}
> 123
123
```
## Lazy data structures
We have so far assumed eager evaluation for data structures (pairs, variants, records). Alternatively though, we could use a lazy evaluation strategy for data structures, e.g. for pairs, we define a pair value to be $(e_1,e_2)$ instead of $(v_1,v_2)$, then only evaluate $e_1$ if it is needed, e.g. $fst(e_1,e_2)$ would then evaluate $e_1$ only.