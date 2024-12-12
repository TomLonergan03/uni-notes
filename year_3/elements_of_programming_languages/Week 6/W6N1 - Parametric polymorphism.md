Polymorphism allows a function to operate on many different types. For a (very simple) example, we have the identity function:
```scala
def idInt(x: Int) = x
def idString(x: String) = x
def idPair(x: (Int, String)) = x
```
This is obviously impractical to define for every possible type, but we cannot just write:
```scala
def id(x) = x
```
as every argument must have a type.
Instead we do:
```scala
def id[A](x: A) = x
```
This says, the function `id[A]` takes something of type `A` and returns something of type `A`. e.g:
```scala
val x: Int = 12;
id[int](x) // returns an int
```

We refer to `A` as a **type variable**.

# Extending LData
We add type variables $A,B,C,...$, type abstractions, type applications, and polymorphic types:
$$
\begin{aligned}
e\;&::=\;...\:|\:\Lambda A.e\:|\:e[\tau]\\
\tau\;&::=\;...\:|\:A\:|\:\forall A.\tau\\
v\;&::=\;...\:|\:\Lambda A.e
\end{aligned}
$$
We use [[W3N1 - Variables and substitution#Substitution|capture avoiding substitution]] of types for type variables in expressions and types.
- The type $\forall A.\tau$ is the type of expressions that can have type $\tau[\tau'/A]$ for any $A$ (which is bound in $\tau$).
- The expression $\Lambda A.e$ introduces a type variable for use in $e$
- The expression $e[\tau]$ instantiates a type abstraction

## Free type variables
As types now have variables, they are also subject to [[W3N2 - Scope|scoping]].
The polymorphic type $\forall A.\tau$ binds $A$ in $\tau$.
We write $FTV(\tau)$ for the free type variables of a type:
$$
\begin{aligned}
FTV(A)\;&=\;\{A\}\\
FTV(\tau_1\times\tau_2)\;&=\;FTV(\tau_1)\cup FTV(\tau_2)\\
FTV(\tau_1+\tau_2)\;&=\;FTV(\tau_1)\cup FTV(\tau_2)\\
FTV(\forall A.\tau)\;&=\;FTV(\tau)-\{A\}\\
FTV(\tau)\;&=\;\emptyset\text{ otherwise}\\
FTV(x_1:\tau_1,...,x_n:\tau_n)\;&=\; FTV(\tau_1)\cup...\cup FTV(\tau_n)
\end{aligned}
$$
[[W3N2 - Scope#Alpha conversion|Alpha-equivalence]] and type substitution are defined similarly to expressions.

## Typechecking polymorphic expressions
$$
\frac{\Gamma\vdash e:\tau\;\;\;\;A\not\in FTV(\Gamma)}{\Gamma\vdash\Lambda A.e:\forall A.\tau}
$$
$$
\frac{\Gamma\vdash e:\forall A.\tau}{\Gamma\vdash e[\tau_0]:\tau[\tau_0/A]}
$$
- $\Lambda A.e$ must typecheck with a parameter $A$ that is not already in use in the context
- $e[\tau_0]$ applies a polymorphic expression to a type. We then determine the type by substituting $A$ with $\tau_0$

## Evaluating polymorphic expressions
$$
\frac{e\Downarrow\Lambda A.e_0\;\;\;\;e_0[\tau/A]\Downarrow v}{e[\tau]\Downarrow v}
$$
$$
\frac{}{\Lambda A.e\Downarrow\Lambda A.e}
$$
In $L_{Poly}$ type information is irrelevant at runtime, but other languages (including Scala) retain some run time type information.

## Syntactic sugar
We can add syntactic sugar for function definitions to allow for type parameters:
$$
\text{let fun }f[A](x:\tau)=e\text{ in }...
$$
which is equivalent to:
$$
\text{let }f=\Lambda A.\lambda x:\tau.e\text{ in }...
$$
In either case, a function call can be written as $f[\tau](x)$

# Other forms of polymorphism
Polymorphism refers to several related techniques of "code reuse":
- [[W5N3 - Subtyping|Subtyping]] is a form of polymorphism based on relations between types
- Parametric polymorphism is abstraction over type parameters
- Ad hoc polymorphism is function *overloading*: the reuse of the same name for multiple implementations, e.g. $+$ is often both numeric addition and string concatenation