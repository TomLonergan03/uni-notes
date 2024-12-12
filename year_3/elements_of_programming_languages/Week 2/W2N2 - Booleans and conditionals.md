We can extend [[W1N2 - Abstract syntax#BNF grammars|LArith]] with equality tests and boolean true/false values. This gives us $L_{If}$.
$$
e::= ...| b\in\mathbb{B}|e==1
$$
We can evaluate $2+2==4$, but can't and/or boolean values, or make decisions based on the results.
If we add an "if then else" operation:
	$$e::=...|\text{ if }e\text{ then }e_1\text{ else }e_2$$
Now, $\text{if true then }1\text{ else }2$ is evaluated to 1

Now, if $\text{if }e\text{ then }e_1\text{ else }e_2$ is the first expression to make a non-trivial choice: whether to evaluate $e_1$ or $e_2$.

We now have a new definition of a value:
$$v::=n\in\mathbb{N}|b\in\mathbb{B}$$
and some more evaluation rules:
$$
\begin{matrix}
		\frac{e_1\Downarrow v\text{ }e_2\Downarrow v}{e_1 == e_2\Downarrow\text{true}} &
		\frac{e_1\Downarrow v_1\text{ }e_2\Downarrow v_2\text{ }v_1\ne v_2}{e_1==e_2\Downarrow\text{false}} \\
		\frac{e\Downarrow\text{true }e_1\Downarrow v_1}{\text{if }e\text{ then }e_1\text{ else }e_2\Downarrow v_1} &
		\frac{e\Downarrow\text{false }e_2\Downarrow v_2}{\text{if }e\text{ then }e_1\text{ else }e_2\Downarrow v_2}
\end{matrix}
$$

and extend the [[W1N2 - Abstract syntax#Scala|interpreter]] to:
```scala
abstract class Expr
case class Num(n: Integer) extends Expr
case class Plus(e1: Expr, e2: Expr) extends Expr
case class Times(e1: Expr, e2: Expr) extends Expr

def size (e: Expr): Int = e match {
	case Num(n) => 1
	case Plus(e1, e2) => size(e1) + size(e2) + 1
	case Times(e1, e2) => size(e1) + size(e2) + 1
}

// New expression forms
case class Bool(n: Boolean) extends Expr
case class Eq(e1: Expr, e2: Expr) extends Expr
case class IfThenElse(e: Expr, e1: Expr, e2: Expr) extends Expr

// Differentiate between types of values
abstract class Value
case class NumV(n: Int) extends Value
case class BoolV(b: Boolean) extends Value

// Helpers for eval
def add(v1: Value, v2: Value): Value = (v1, v2) match {
	case (NumV(v1), NumV(v2)) => NumV(v1 + v2)
	case _ => sys.error("Can only add numbers")
}

def mult(v1: Value, v2: Value): Value = (v1, v2) match {
	case (NumV(v1), NumV(v2)) => NumV(v1 * v2)
	case _ => sys.error("Can only multiply numbers")
}

def eq(v1: Value, v2: Value): Value = (v1, v2) match {
	case (NumV(n1), NumV(n2)) => BoolV(n1 == n2)
	case (BoolV(b1), BoolV(b2)) => BoolV(b1 == b2)
	case _ => sys.error("Can compare numbers to numbers or bools to bools")
}

// Eval
def eval(e: Expr): Value = e match {
	case Num(n) => NumV(n)
	case Plus(e1, e2) => add(eval(e1), eval(e2))
	case Times(e1, e2) => mul(eval(e1), eval(e2))
	case Bool(b) => BoolV(b)
	case Eq(e1, e2) => eq(eval(e1), eval(e2))
	case IfThenElse(e, e1, e2) => eval(e) match {
		case BoolV(true) => eval(e1)
		case BoolV(false) => eval(e2)
	}
}
```
