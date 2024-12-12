We have considered [[W4N2 - Variant types#Optionals|several]] [[W4N2 - Variant types#OK or error type|approaches]] to error handling. Exceptions are another popular approach (used in Java, C++, Scala, Python, etc.), where a `throw e` statement raises an exception `e`, and a `try { ... } catch (Exception e) { ... }` block which executes some code, and if a specific exception occurs then some other code is executed, and the exception stops propagating. There is often also a `finally` clause, which occurs after a try catch block and is always executed, regardless of if an exception is thrown.
# Exceptions for shortcutting
It is possible, but unusual, to use exceptions for "normal" computations, e.g:
```scala
def product(l: List[Int]) = {
	object Zero extends Throwable
	def go(l: List[Int]): Int = l match {
		case Nil => 1
		case x::xs =>
			if (x == 0) { throw Zero } else { x * go(xs) }
	}
	try { go(1) }
	catch { case Zero => 0 }
}
```
(although in this case we could also just do `if (x==0) { 0 } else ...`)
# Exceptions in practice
## Java
In Java, potentially unhandled exceptions generally (bar some special cases such as `NullPointerException`) need to be declared in the types of methods, e.g:
```java
void writeFile(String filename) throws IOException { ... }
```
Failing to handle or declare an exception is a type error.
Exceptions are subclasses of `java.lang.Throwable`, and multiple catch blocks may be used.
## Scala
Scala has a similar mechanism, except the `catch` block is a pattern match on exceptions, and does not require a `throws` clause (which can be a problem, but simplifies typing for higher order functions).

# Modelling exceptions
We can extend [[W3N4 - Functions#Recursive functions|L_Rec]] with a model of exceptions:
$$
e::=\cdots\:|\:\text{raise }e\:|\:e_1\text{ handle }\{x\Rightarrow e_2\}
$$
Here, $\text{raise }e$ throws an arbitrary (practically we would probably want to restrict this) value, and $e_1\text{ handle }\{x\Rightarrow e_2\}$ evaluates $e_1$, and if an exception is thrown during evaluation, binds the value $v$ it to $x$ and evaluates $e_2$
## Exceptions and types
Exception constructs are straightforward to typecheck:
$$
\tau::=\cdots\:|\:\text{exn}
$$
Usually, the $\text{exn}$ type is extensible, e.g. by [[W7N1 - Objects and classes#Inheritance|subclassing]].

The typing rules for $L_{Exn}$ are:
$$
\begin{gathered}
\frac{\Gamma\vdash e:\text{exn}}{\Gamma\vdash e:\tau}&
\frac{\gamma\vdash e_1:\tau\;\;\;\;\Gamma,x:\text{exn}\vdash e_2:\tau}{\Gamma\vdash e_1\text{ handle }\{x\Rightarrow e_2\}:\tau}
\end{gathered}
$$
Note that $\text{raise }e$ can have any type, as it never returns.
## Semantics of exceptions
To formalise the semantics of exceptions, we need an additional judgement $e\text{ raises }v$, which says that expression $e$ does not finish, instead raising an exception value $v$.
In [[W9N1 - Small-step semantics|small-step semantics]], raising an exception is expressed as:
$$
\begin{gathered}
\frac{}{\text{raise }v\text{ raises }v}&
\frac{e_1\text{ raises }v}{e_1\oplus e_2\text{ raises }v}&
\frac{e_2\text{ raises }v}{v_1\oplus e_2\text{ raises }v}&
\cdots
\end{gathered}
$$
The first rule is the most interesting one, while the rest are administrative as every expression getting evaluated can potentially raise an exception.

We also need to define semantics for handling an exception:
$$
\begin{gathered}
\frac{e_1\rightarrowtail e'_1}{e_1\text{ handle }\{x\Rightarrow e_2\}\rightarrowtail e'_1\text{ handle }\{x\Rightarrow e_2\}}\\
\frac{}{v_1\text{ handle }\{x\Rightarrow e_2\}\rightarrowtail v_1}\\
\frac{e_1\text{ raises }v}{e_1\text{ handle }\{x\Rightarrow e_2\}\rightarrowtail e_2[v/x]}
\end{gathered}
$$
i.e. if $e_1$ steps normally to $e'_1$, take that step. If $e_1$ raises an exception $v$, substitute it in for $x$ and evaluate $e_2$.
## Interpreting exceptions
We can extend our Scala interpreter for $L_{Rec}$ to manage exceptions as follows:
```scala
case class ExceptionV(v: Value) extends Throwable

def eval(e: Expr): Value = e match {
	...
	case Raise(e: Expr) => throw (ExceptionV(eval(e)))
	case Handle(e1: Expr, x: Variable, e2: Expr) =>
		try {
			eval(e1)
		} catch (ExceptionV(v)) {
			eval(subst(e2, v, x))
		}
}
```
This is a little circular, however it is common to reuse parent language features in an interpreter for a new language (we have also done this for everything else on a similar/lower level, e.g. `Plus(NumV(5), NumV(3))` ultimately uses Scala's definition of `+`!)