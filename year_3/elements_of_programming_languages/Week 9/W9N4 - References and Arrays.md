In [[W8N1 - Imperative programming#$L_{While}$|L_While]], all variables are mutable and global. While useful, mutability and especially global variables make it easier to make mistakes or difficult to understand code. 
We will now look to add mutable state side-effects to [[W6N1 - Parametric polymorphism#Extending LData|L_Poly]], along with some imperative features.
# $L_{Ref}$
We extend $L_{Poly}$ with the following:
$$
\begin{aligned}
e\;&::=\;...\:|\:\text{ref}(e)\:|\:!e\:|\:e_1::=e_2\:|e_1;e_2\\
\tau\;&::=\;...\:|\:\text{ref}[\tau]
\end{aligned}
$$

- $\text{ref}(e)$ evaluates $e$ to $v$ and creates a new reference cell containing $v$.
- $!e$ evaluates $e$ to a reference and looks up its value (dereferences it)
- $e_1:=e_2$ evaluates $e_1$ to a reference cell and $e_2$ to a value then assigns the value to the reference cell
- $e_1;e_2$ evaluates $e_1$, ignores the value, then evaluates $e_2$
# Typing
$$
\begin{gathered}
\frac{\Gamma\vdash e:\tau}{\Gamma\vdash\text{ref}(e):\text{ref}[\tau]}&
\frac{\Gamma\vdash e:\text{ref}[\tau]}{\Gamma\vdash!e:\tau}\\
\frac{\Gamma\vdash e:\text{ref}[\tau]\;\;\;\;\Gamma\vdash e_2:\tau}{\Gamma\vdash e_1:=e_2:\text{unit}}&
\frac{\Gamma\vdash e_1:\tau'\;\;\;\;\Gamma\vdash e_2:\tau}{\Gamma\vdash e_1;e_2:\tau}
\end{gathered}
$$
- $\text{ref}(e)$ creates a reference of type $\tau$ if $e:\tau$
- $!e$ gets a value of type $\tau$ if $e:\text{ref}[tau]$
- $e_1:=e_2$ updates reference $e_1:\text{ref}[\tau]$ with value $e_2:\tau$, and returns $()$
- $e_1;e_2$ evaluates $e_1$, ignores the resultant value, and evaluates $e_2$
# References in Scala
We can define a similar type in Scala:
```scala
class Ref[A](val x: A) {
	private var a = x
	def get = a
	def set(y: A) = { a = y }
}
```
```
scala> val x = new Ref[Int](1)
x: Ref[Int] = Ref@725bef66
scala> x.get
res3: Int = 1
scala> x.set(12)
scala> x.get
res5: Int = 12
```

Once we add references to a functional language, we can use function definitions to define [[W8N1 - Imperative programming#Procedures|procedures]]. Basically, a procedure is just a function with a return type $Unit$, e.g.
```scala
val x = new Ref(42)
def incrBy(n: Int): Unit = {
	x.set(x.get + n)
}
```

A procedure does not return a value, and is only executed for its "side effects" on references. Using the same idea, we can embed all of the constructs of $L_{While}$ in $L_{Ref}$.

# Semantics
Here, we will use the [[W9N1 - Small-step semantics|small step semantics]] we encountered earlier. Here, $\sigma,e\rightarrowtail\sigma'e'$ "in initial state $\sigma$, expression $e$ can step to $e'$ with state $\sigma'$ "
$Ref(e)$ evaluates to a pointer or memory cell location $l\in Loc$
$$
v\;\;::=\;\;...\:|\:l
$$
These special values only appear during evaluation.
$$
\frac{l\not\in locs(\sigma)}{\sigma,\text{ref}(v)\rightarrowtail\sigma[l:=v],l}
$$
$$
\begin{gathered}
\frac{}{\sigma,!l\rightarrowtail\sigma,\sigma(l)}&
\frac{}{\sigma,l\rightarrowtail\sigma[l:=v,()}&
\end{gathered}
$$
We also must update all existing small step rules to pass $\sigma$ around, and account for any side effects which modify it, e.g.
$$
\begin{gathered}
\frac{\sigma,e_1\rightarrowtail\sigma',e'_1}{\sigma,e_1\oplus e_2\rightarrowtail\sigma',e'_1\oplus e_2}&
\frac{}{\sigma,v_1+v_2\rightarrowtail\sigma,v_1+_\mathbb{N}v_2}
\end{gathered}
$$
and so on for every other case (taking care as to which steps are capable of altering state)

And finally, we need rules to evaluate the reference constructs themselves:
$$
\begin{gathered}
\frac{\sigma,e\rightarrowtail\sigma',e'}{\sigma,\text{ref}(e)\rightarrowtail\sigma',\text{ref}(e')}&
\frac{\sigma,e\rightarrowtail\sigma',e'}{\sigma,!e\rightarrowtail\sigma',!e'}\\
\frac{\sigma,e_1\rightarrowtail\sigma',e'_1}{\sigma,e_1:=e_2\rightarrowtail\sigma',e'_1:=e_2}&
\frac{\sigma,e_2\rightarrowtail\sigma',e'_2}{\sigma,v_1:=e_2\rightarrowtail\sigma',v_1:=e'_2}&
\end{gathered}
$$
Again, we need to allow for updates to $\sigma$, e.g. if we evaluate $\text{ref}(\text{ref}(42))$
Example evaluation:
$$
\begin{aligned}
&\text{let }r=\text{ref}(42)\text{ in }r:=17;!r\\
\rightarrowtail\;\;&[l:=42],\text{let }r=l\text{ in }r:=7;!r\\
\rightarrowtail\;\;&[l:=42],l:=17;!l\\
\rightarrowtail\;\;&[l:=17],!l\\
\rightarrowtail\;\;&[l:=17],17
\end{aligned}
$$
We don't need a rule for $e_1;e_2$, even though they would be straightforward. Instead we can say:
$$
e_1;e_2\;\Longleftrightarrow\;\text{let }\_=e_1\text{ in }e_2
$$
where $\_$ stands for any variable that is not used in $e_1$ or $e_2$, meaning that all that remains from $e_1$ is it's side effects on the state $\sigma$.

Now, evaluation order matters, e.g.
$$
\text{let }r=\text{ref}(1)\text{ in }(r:=1000;3)+!r
$$
here, we get either $4$ or $1003$ depending on if we evaluate left-to-right or right-to-left:
- Left-to-right: $r:=1000$ is evaluated before $!r$, so $3+1000=1003$
- Right-to-left: $!r=1$, then LHS is evaluated to 3, so $3+1=4$

This is alright however, as our small step rules always evaluate left before right

# Arrays
Arrays generalise references to allow getting and setting by index:
$$
\begin{aligned}
e\;&::=\;...\:|\:\text{array}(e_1,e_2)\:|\:e_1[e_2]\:|\:e_1[e_2]:=e_3\\
\tau\;&::=\;...\:|\:\text{array}[\tau]
\end{aligned}
$$
- $\text{array}(n,init)$ creates an array of $n$ elements, initialised to $init$
- $arr[i]$ gets the $i\text{th}$ element; $arr[i]:=v$ sets the $i\text{th}$ element to $v$
- This introduces the potential problem of out-of-bounds accesses, which are often a [[W8N3 - Buffer overruns|security risk]]
# References and subtyping
If `Integer <: Object` and `String <: Object`, and we allow [[W5N3 - Subtyping#Subtyping for functions|contravariant subtyping]] for `Ref` i.e. `Ref[-A]`
We can then do the following:
```scala
val x: Ref[Object] = new Ref(new Integer(42))
// String <: Object, so Ref[Object] <: Ref[String]
x.get.length() // unsound, Int has no length
```

If we instead allow covariant subtyping (`Ref[+A]`) there are still possibilities for unsound behaviour:
```scala
val x: Ref[String] = new Ref(new String("asdf"))
def bad(y: Ref[Object]) = y.set(new Integer(42))
bad(x) // x keeps type Ref[String]
x.get.length() // unsound
```

Therefore, mutable parameterised types like `Ref` must be invariant (neither co or contravariant)

(Java got this wrong for built in array types)