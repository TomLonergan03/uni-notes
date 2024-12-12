While pairs allow us to combine two types, we might want a value that is either one or the other type.

# Optionals
Sometimes we want to produce either a regular value or a special null value.
This occurs in many types in languages like Java (`NullPointerException`), or as a special type such as `Option<T>` in Rust (which can be `Some(T)` or `None`) or `std::optional<T>` in C++ (`std::optional<T>` or `std::nullopt`).

## Typing for optionals
$$
\begin{align}
&e::=...|\text{none}|\text{some}(e)\\
&v::=...|\text{option}[t]
\end{align}
$$
$$
\begin{matrix}
\frac{}{\Gamma\vdash\text{none}:\text{option}[\tau]}&
\frac{\Gamma\vdash e:\tau}{\Gamma\vdash\text{some}(e):\text{option}[\tau]}
\end{matrix}
$$
We can use $\text{none}$ to indicate absence of a value, and $\text{some}(e)$ to give a value. In addition, the type of the expression tells us if null values are possible.
## Error codes
Where an option returns a `none`, it often indicates a failure has occurred. In this case, we might want to get more information on why the failure occurred.

In older languages such as C, special return values are used for errors, commonly `-1`. The actual error code is passed via a global variable, often `errno`.
# OK or error type
If we want to return either a normal value $\tau_{ok}$ or and error type $\tau_{err}$.
$$
\begin{aligned}
e&::=...|\text{ok}(e)|\text{err}(e)\\
\tau&::=...|okOrErr[\tau_1\tau_2]
\end{aligned}
$$
Basically, if $e$ has type $\tau_{ok}$ then $\text{ok}(e)$ has the type $\text{okOrErr}[\tau_{ok},\tau_{err}]$.
If $e$ has type $\tau_{err}$ then $\text{err}(e)$ has the type $\text{okOrErr}[\tau_{ok},\tau_{err}]$.
### Using OK-or-error
If we have an $\text{okOrErr}[\tau_{ok},\tau_{err}]$ then we will want to branch on its value:
- If $v$ is $\text{ok}(v_{ok})$, we will want to extract $v_{ok}$ and continue computation
- If $v$ is $\text{err}(v_{err})$, we will want to extract $v_{err}$ and maybe log it/recover from that error/halt computation
## Case analysis
We consider a case analysis construct as follows:
$$
\text{case }e\text{ of }\{\text{ok}(x)\Rightarrow e_{ok};\;\text{err}(y)\Rightarrow e_{err}\}
$$
This is a generalised conditional: "If $e$ evaluates to $\text{ok}(v_{ok})$ then evaluate $e_{ok}$ with $v_{ok}$ replacing $x$, otherwise evaluate $e_{err}$ with $v_{err}$ replacing $y$."
Here, $x$ is bound in $e_{ok}$ and $y$ is bound in $e_{err}$
This is familiar from Scala:
```scala
e match {
	case Ok(x) => e1
	case Err(x) => e2
}
```

# Generalised variants
In both above variant types, the $\text{ok}$ and $\text{err}$ cases are symmetric (in the case of [[W4N2 - Variant types#Optionals|optionals]] the $\text{none}$ case being treated as $\text{none}(\text{unit})$). Variants are also useful for situations other than error handling, so we can rename and generalise the notion.
$$
\begin{aligned}
e&::=...|\text{left}(e)|\text{right}(e)\\&|\text{case }e\text{ of }\{\text{left}(x)\Rightarrow e_1\; ;\text{right}(y)\Rightarrow e_2\}\\
v&::=...|\text{left}(v)|\text{right}(v)\\
\tau&::=...|\tau_1+\tau_2
\end{aligned}
$$
The type $\tau_1+\tau_2$ is a **variant type** (or a **sum** or **disjoint union**)
## Typing
$$
\begin{matrix}
\frac{\Gamma\vdash e:\tau_1}{\Gamma\vdash\text{left}(e):\tau_1+\tau_2}&
\frac{\Gamma\vdash e:\tau_2}{\Gamma\vdash\text{right}(e):\tau_1+\tau_2}
\end{matrix}
$$
$$
\frac{\Gamma\vdash e:\tau_1+\tau_2\;\;\;\;\Gamma,x:\tau_1\vdash e_1:\tau\;\;\;\;\Gamma,y:\tau_2\vdash e_2:\tau}{\Gamma\vdash\text{case }e\text{ of }\{\text{left}(x)\Leftarrow e_1;\;\text{right}(y)\Rightarrow e_2\}:\tau}
$$
Here, $\text{left}$ and $\text{right}$ "wrap" $\tau_1$ or $\tau_2$ as $\tau_1+\tau_2$
Case is similar to a [[W2N2 - Booleans and conditionals|conditional]] except we can use the value extracted from $\text{left}(v)$ or $\text{right}(v)$
## Semantics
$$
\begin{matrix}
\frac{e\Downarrow v}{\text{left}(e)\Downarrow\text{left}(v)}&
\frac{e\Downarrow v}{\text{right}(e)\Downarrow\text{right}(v)}
\end{matrix}
$$
$$
\frac{e\Downarrow\text{left}(v_1)\;\;\;\;e_1[v_1,x]\Downarrow v}{\text{case }e\text{ of }\{\text{left}(x)\Leftarrow e_1;\;\text{right}(y)\Rightarrow e_2\}\Downarrow v}
$$
$$
\frac{e\Downarrow\text{right}(v_2)\;\;\;\;e_2[v_2,y]\Downarrow v}{\text{case }e\text{ of }\{\text{left}(x)\Leftarrow e_1;\;\text{right}(y)\Rightarrow e_2\}\Downarrow v}
$$

# Defining Booleans and options
The boolean type $\text{bool}$ can be defined as $\text{unit}+\text{unit}$
$$
\begin{matrix}
\text{true}\;\Longleftrightarrow\;\text{left}()&
\text{false}\;\Longleftrightarrow\;\text{right}()
\end{matrix}
$$
We can then define conditionals as case analysis, and just ignore the variables:
$$
\begin{aligned}
\text{if }e&\text{ then }e_1\text{ else }e_2\\
&\Longleftrightarrow\text{case }e\text{ of }\{\text{left}(x)\Rightarrow e_1;\;\text{right}(y)\Rightarrow e_2\}
\end{aligned}
$$
Likewise, the option type is definable as $\tau+\text{unit}$:
$$
\begin{matrix}
\text{some}(e)\;\Longleftrightarrow\;\text{left}(e)&
\text{none}\;\Longleftrightarrow\;\text{right}()
\end{matrix}
$$
