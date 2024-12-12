So far, our [[W1N2 - Abstract syntax|ASTs]] only consider pure computations, where once a value is bound to a variable, that value is immutable.
This is different to how most programming languages treat variables, where they are mutable by default. Only a few languages, like Haskell, are truly "pure". Other languages often strike a balance, e.g. Scala has both mutable (`var`) and immutable (`val`) variables, while Rust has immutable `let` by default and `let mut` to declare a mutable variable.

# Mutable vs immutable
Advantages of immutability:
- Programs are easier to reason about and optimise
- Typing makes it clearer as to what a program can/cannot do
Advantages of mutability:
- Some common data structures are easier to implement
- Easier to translate to performant machine code
- Nicely ties in to OOP model of objects with hidden mutable state and public methods

# $L_{While}$
$L_{While}$ is a language similar to [[W2N2 - Booleans and conditionals|LIf]] which includes statements:
$$
\begin{aligned}
Stmt\ni s\:::=\:&\text{skip}\;|\;s1;s2\;|\;x:=e\\
&|\;\text{if }e\text{ then }s_1\text{ else }s_2\\
&|\;\text{while }e\text{ do }s
\end{aligned}
$$
- $\text{skip}$ does nothing
- $s_1;s_2$ does $s_1$, then $s_2$
- $x:=e$ evaluates $e$ and assigns the value to $x$
- $\text{if }e\text{ then }s_1\text{ else }s_2$ evaluates $e$, then evaluates $s_1$ or $s_2$ based on the result
- $\text{while }e\text{ do }s$ tests $e$. If true, it evaluates $s$ and loops, otherwise stops.

n.b. all variables in $L_{While}$ are mutable.

# Interpreter
Here, we are defining a pure interpreter, where each operation has no side effects:
```scala
def exec(env: Env[Value], s: Stmt): Env[Value] =
s match {
	case Skip => env
	case Seq(s1, s2) =>
		val env1 = exec(env, s1)
		exec(env1, s2)
	case IfThenElseS(e, s1, s2) => eval(env, e) match {
		case BoolV(true) => exec(env, s1)
		case BoolV(false) => exec(env, s2)
	}
	case WhileDo(e, s) => eval(env, e) match {
		case BoolV(true) =>
			val env1 = exec(env, s)
			exec(env1, WhileDo(e, s))
		case BoolV(false) => env
	}
	case Assign(x, e) => 
		val v = eval(env, e)
		env + (x -> v)
}
```

# Evaluation
Here, we use 3 components: initial state $\sigma$, statement $s$, and resultant state $\sigma'$:
$$
\begin{gathered}
\frac{}{\sigma,\text{skip}\Downarrow \sigma} & 
\frac{\sigma,s_1\Downarrow\sigma'\;\;\;\;\sigma',s_2\Downarrow\sigma''}{\sigma,s_1;s_2\Downarrow\sigma''}\\
\frac{\sigma,e\Downarrow\text{true}\;\;\;\;\sigma,s_1\Downarrow\sigma'}{\sigma,\text{if }e\text{ then }s_1\text{ else }s_2\Downarrow\sigma'}&
\frac{\sigma,e\Downarrow\text{false}\;\;\;\;\sigma,s_2\Downarrow\sigma'}{\sigma,\text{if }e\text{ then }s_1\text{ else }s_2\Downarrow\sigma'}\\
\frac{\sigma,e\Downarrow\text{true}\;\;\;\;\sigma,s\Downarrow\sigma'\;\;\;\;\sigma',\text{while }e\text{ do }s\Downarrow\sigma''}{\sigma,\text{while }e\text{ do }s\Downarrow\sigma''}&
\frac{\sigma,e\Downarrow\text{false}}{\text{while }e\text{ do }s\Downarrow\sigma}\\
\frac{\sigma,e\Downarrow v}{\sigma,x:=e\Downarrow[x:=v]}
\end{gathered}
$$

We can also define some other common control flow constructs in terms of these, e.g:
$$
\begin{aligned}
\text{if }e\text{ then }s\;\;&\Longleftrightarrow\;\;\text{if }e\text{ then }s\text{ else skip}\\
\text{do }s\text{ while }e\;\;&\Longleftrightarrow\;\;s;\text{ while }e\text{ do }s\\
\text{for }(i\in n..m)\;\;&\Longleftrightarrow\;\;i:=n;\text{ while }i\leq m\text{ do }\{s;i=i+1\}
\end{aligned}
$$
# Procedures
$L_{While}$ is lacking, among other things, procedures (aka function calls). Procedures can be added to $L_{While}$ much like functions in [[W3N4 - Functions|LRec]], but instead we will combine $L_{While}$ and $L_{Rec}$ later.