# Named functions
A simple way to add support for functions is as follows:
$$
e::=...|f(e)|\text{let fun }f(x:\tau)=e_1\text{ in }e_2
$$

Which means: define a function $f$ which takes an argument $x$ and whose result is the expression $e1$.

This is pretty limited:
- Only takes 1 argument
- No recursion
- Functions are not first-class "values": we can only call $f$, but can't pass it as an argument to another

E.g.
$$
\text{let fun }square(x:int)=x\times x\text{ in }...
$$

## Types for named functions
A **type constructor** $\tau_1\rightarrow\tau_2$ means "the type of functions taking arguments in $\tau_1$ and returning $\tau_2$".

We can typecheck named functions as follows:
$$
\begin{matrix}
\frac{\Gamma,x:\tau_1\vdash e_1:\tau_2\;\;\;\;\Gamma,f:\tau_1\rightarrow\tau_2\vdash e_2:\tau}{\Gamma\vdash\text{let fun }f(x:\tau_1)=e_1\text{ in }e_2:\tau}\\
\frac{\Gamma(f)=\tau_1\rightarrow\tau_2\;\;\;\;\Gamma\vdash e:\tau_1}{\Gamma\vdash f(e):\tau_2}
\end{matrix}
$$
For convenience, we can use the same environment $\Gamma$ for both variable types and function types.
## Semantics of named functions
We need an environment mapping function names $f$ to their "definitions", written as $\delta$.
When we encounter a function definition, we add it to $\delta$ as so:
$$
\frac{\delta[f\rightarrow\langle x\Rightarrow e_1\rangle],e_2\Downarrow v}{\delta,\text{ let fun }f(x:\tau)=e_1\text{ in }e_2\Downarrow v}
$$
When we encounter a function call, we look up the definition, then evaluate the body with the argument value substituted for the argument:
$$
\frac{\delta,e_0\Downarrow v_0\;\;\;\;\delta(f)=\langle x\Rightarrow e\rangle\;\;\;\;\delta,e[v_0/x]\Downarrow v}{\delta,f(e_0)\Downarrow v
}
$$
## Static vs dynamic scope
In **static scope**, the scope and binding occurrences of all variables can be determined from the program text only, without evaluating the program.

In **dynamic scope**, this is not necessarily the case, and scope can vary depending on the context in which it is evaluated at run time.

E.g.
$$
\begin{align}
&\text{let }x=1\text{ in}\\
&\text{let fun }f(y:\text{int})=x+y\text{ in}\\
&\text{let }x=10\text{ in }f(3)
\end{align}
$$

This behaves differently in each approach, as $x$ is bound to $1$ when $f$ is defined, but $x$ is bound to $10$ when $f$ is called

### Dynamic scope breaks type soundness
$$
\begin{align}
&\text{let }x=1\text{ in}\\
&\text{let fun }f(y:\text{int})=x+y\text{ in}\\
&\text{let }x=\text{true in }f(3)
\end{align}
$$
Here, when we typecheck $f$, $x$ is an integer, but it is re-bound to a boolean by the time $f$ is called.
The program as a whole typechecks, but we get a run-time type error.

Dynamic scope is arguably useful in an untyped/dynamically typed language, but is a bad idea in statically typed languages.

# Anonymous functions
In many languages we can define an expression for a function without a name:
$$
\lambda:\tau.e
$$
Here, $\lambda$ introduces an anonymous function in which $x$ is bound in $e$.

We can extend $L_{Lam}$ to be $L_{Let}$ extended with typed $\lambda$-abstraction and application as follows:
$$
\begin{align}
&e::=...|e_1\;e_2|\lambda x:\tau.e\\
&\tau::=...|\tau_1\rightarrow\tau_2
\end{align}
$$
## Types for lambdas
Our new type, $\tau_1\rightarrow\tau_2$ is the type of functions from $\tau_1$ to $\tau_2$.
We can extend the typing rules as follows:
$$
\begin{matrix}
\frac{\Gamma,x:\tau_1\vdash e:\tau_2}{\Gamma\vdash\lambda x:\tau_1.e:\tau_1\rightarrow\tau_2}&
\frac{\Gamma\vdash e_1:\tau_1\rightarrow\tau_2\;\;\;\;e_2:\tau_1}{\Gamma\vdash e_1 e_2:\tau_2}
\end{matrix}
$$

## Evaluation for lambdas
We can extend the evaluation rules to include our new possibilities for $e$:
$$
\begin{matrix}
\frac{}{\lambda x.e\Downarrow\lambda x.e}&
\frac{e_1\Downarrow\lambda x.e\;\;\;\;e_2\Downarrow v_2\;\;\;\;e[v_2/x]\Downarrow v}{e_1\;e_2\Downarrow v}
\end{matrix}
$$

At this point we don't need a unique $\text{let fun}$ anymore, it can just be defined as a $\text{let}$ binding for a $\lambda$ function:
$$
\text{let fun }f(x:\tau)=e_1\text{ in }e_2\Leftrightarrow\text{ let }f=\lambda x:\tau.e_1\text{ in }e_2
$$

# Recursive functions
$L_{Lam}$ still cannot express general recursion.

## Named recursive functions
In many languages, named function definitions are recursive by default. (C, Python, Java, Haskell, Scala).
Others explicitly distinguish between nonrecursive and recursive function definitions. (Scheme, OCaml, F#).
E.g. in OCaml:
```OCaml
let f x = e (* nonrecursive, only x is in scope in e *)
let rec f x = e (* recursive, both f and x are in scope in e e*)
```

## Anonymous recursive function
Inspired by $L_{Lam}$ we can introduce a notation for anonymous recursive functions:
$$
e::=...|\text{ rec }f(x:\tau_1):\tau_2.e
$$
Here, $f$ is a local name for the function which is in scope only within that function in $e$.
We define $L_{Rec}$ to be $L_{Lam}$ extended with $\text{rec}$.
Now, we can define $\text{let rec}$ as syntactic sugar:
$$
\begin{align}
&\text{let rec }f(x:\tau_1):\tau_2=e_1\text{ in }e_2\\
&\Longleftrightarrow\text{let }f=\text{rec }f(x:\tau_1):\tau_2.e_1\text{ in }e_2
\end{align}
$$
Note that here, the $f$ in $\text{let }f$ is in scope in $e_2$, while the $\text{rec }f$ is in scope within $e_1$. Regardless, both cases of $f$ evaluate to the same thing.
### Typing
The types of $L_{Rec}$ are the same as in $L_{Lam}$, with just one extra rule for typechecking:
$$
\frac{\Gamma,f:\tau_1\rightarrow\tau_2,x:\tau_1\vdash e:\tau_2}{\Gamma\vdash\text{rec }f(x:\tau_1):\tau_2.e:\tau_1\rightarrow\tau_2}
$$
This says: to typecheck a recursive function,
- bind $f$ to the type $\tau_1\rightarrow\tau_2$ (so that it can be used in checking $e$)
- bind $x$ to the type $\tau_1$ (so that it can be used as an argument in $e$)
- then typecheck $e$.
### Semantics
Like a $\lambda$-term, a recursive function is a value:
$$
v::=...|\text{ rec }f(x).e
$$
We can evaluate recursive functions as follows:
$$
\begin{matrix}
\frac{}{\text{rec }f(x).e\Downarrow\text{ rec }f(x.e)}\\
\frac{e_1\Downarrow\text{rec }f(x).e\;\;\;\;e_2\Downarrow v_2\;\;\;\;e[\text{rec }f(x).e/f,v_2/x]\Downarrow v}{e_1\;e_2\Downarrow v}
\end{matrix}
$$

At this point, $L_{Rec}$ is Turing complete, meaning it can compute any computable function. Despite this, it is still so limited it is not very useful as a general purpose language.
## Mutual recursion
Defining a mutually recursive function, such as:
```scala
def even(n: Int) = if n == 0 then true else odd(n - 1)
def odd(n: Int) = if n == 0 then false else even(n - 1)
```
is not possible in $L_{Rec}$, and is not simple to add support for.

One solution is to generalise $\text{let rec}$ to take an arbitrary number of functions, all of which should be in scope in $e$:
$$
\text{let rec }f_1(x_1:\tau_1):\tau'_2=e_1\text{ and }...\text{ and }f_n(x_n:\tau_n):\tau'_n=e_n\text{ in }e
$$
This gets messy fast, and other solutions will be revisited later.