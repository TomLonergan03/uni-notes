# Records
**Records** generalise [[W4N1 - Pairs|pairs]] to $n$-tuples with named fields
$$
\begin{align}
e\;&::=\;...\:|\:\langle l_1=e_1,...,l_n=e_n\rangle\:|\:e.l\\
v\;&::=\;...\:|\:\langle l_1=v_1,...,l_n=v_n\rangle\\
\tau\;&::=\;...\:|\:\langle l_1=\tau_1,...,l_n=\tau_n\rangle\\
\end{align}
$$
Examples:
$$\langle fst=1,snd=\text{"forty-two"}\rangle.snd\rightarrow\text{"forty-two"}
$$
Record fields can also be first-class functions:
$$
\langle x=3.0,y=4.0,length=\lambda(x,y).sqrt(x*x+y*y)\rangle
$$

# Named variants
**Named variants** generalise [[W4N2 - Variant types|binary variants]] just as records generalise pairs.
$$
\begin{align}
e\;&::=\;...\:|\:C_i(e)\:|\:\text{case }e\text{ of }\{C_1(x)\Rightarrow e_1;...;C_n(x)\Rightarrow e_n\}\\
v\;&::=\;...\:|\:C_i(v)\\
\tau\;&::=\;...\:|\:[C_1:\tau_1,...,C_n:\tau_n]\\
\end{align}
$$
## In Scala
Scala uses the `case class` mechanism for named variants, e.g:
```scala
abstract class IntList
case class Nil() extends IntList
case class Cons(head: Int, tail: IntList) extends IntList
```
and we can pattern match on those classes:
```scala
def map(f: Int => Int, x: IntList) = x match {
	case Nil() => Nil()
	case Cons(head, tail) => Cons(f(head), map(f, tail))
}
```
Variables cannot be repeated, instead explicit equality tests must be used:
```scala
// This does not only match if both elements in the list are equal
x match {
	case Cons(x, Cons(x, Nil())) => ...
}

// This must be done instead
x match {
	case Cons(x, Cons(y, Nil())) => if x == y then ... else ...
}
```
and the special pattern `_` matches everything and patterns can overlap and are tested in order:
```scala
result match {
	case OK => println("All is well")
	case _ => println("There is an error")
}
// is not the same as
result match {
	case _ => println("There is an error")
	case OK => println("All is well")
}
```
The patterns that are matched can also be nested:
```scala
l match {
	case Cons(x, Cons(y, Nil())) => ...
}
```
but this can also be expanded to:
```scala
l match {
	case Cons(x, t1) => t1 match {
		case Cons(y, t2) => t2 match {
			case Nil() => ...
		}
	}
}
```
