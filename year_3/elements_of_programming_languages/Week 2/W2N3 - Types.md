With [[W2N2 - Booleans and conditionals|LIf]], we can now write expressions such as 
$$ e_1 = 1+(2==3)$$
or
$$e_2=if1then2else3$$
What should these expressions evaluate to?
There are no value $v$ such that $e_1\Downarrow v$ or $e_2\Downarrow v$, therefore the [[W2N1 - Evaluation#Totality and structural induction|totality]] of $L_{Arith}$ no longer holds.

In some languages, e.g. C/Java/JS, there are built in conversion rules.
E.g. if an integer is needed and a boolean is available, convert true to 1 and false to 0.
This is convenient but can make program's behaviours unpredictable.

We can use a type system to define a subset of "well-formed" programs.

In the case of $L_{If}$, this means that:
- arguments to arithmetic operations and equality tests should be numeric values
- arguments to conditional tests should be Boolean values

# Types, informally
Given an expression $e$:
- If $e=n$ then $e$ has type "integer"
- If $e=e_1+e_2$ then $e_1$ and $e_2$ must have type "integer". If so, $e$ has type "integer" else it is a type error
- If $e=e_1\times e_2$ then similar
- If $e=\text{true}|\text{false}$ then $e$ has type "boolean"
- If $e=e_1==e_2$ then $e_1$ and $e_2$ must have the same type. If so, $e$ has type "boolean", otherwise it is a type error
- If $e=\text{if }e_0\text{ then }e_1\text{ else }e_2$ then $e_0$ must have type "boolean", and $e_1$ and $e_2$ must have the same type. If so, $e$ has the same type as $e_1$ and $e_2$, else, type error.
# Types, formally
We can define the possible types using a [[W1N2 - Abstract syntax#BNF grammars|BNF grammar]]:
$$\text{Type}\ni\tau::=\text{ int }|\text{ bool}$$
We use the notation $\vdash e:\\tau$ to say $e$ is a well-formed term of type $\tau$ (or "$e$ has type $\tau$")
We can formally define the above types as so:
$$
\begin{matrix}
	\frac{n\in\mathbb{N}}{\vdash n: \text{int}} &
	\frac{\vdash e_1: \text{int }\vdash e_2:\text{int}}{\vdash e_1 + e_2: \text{int}}\\
	
	\frac{\vdash e_1: \text{int }\vdash e_2:\text{int}}{\vdash e_1 \times e_2: \text{int}} &
	\frac{b\in\mathbb{B}}{\vdash b:\text{bool}}\\

	\frac{\vdash e_1:\tau\text{ }\vdash e_2:\tau}{\vdash e_1==e_2:\text{bool}} &
	\frac{\vdash e :\text{bool }\vdash e_1:\tau\text{ }\vdash e_2:\tau}{\vdash \text{if }e\text{ then }e_1\text{ else }e_2:\tau}
\end{matrix}
$$
Subexpressions of == must be of equal types as they both use the same $\tau$, similarly for conditionals
# Soundness
If an expression is well typed, then it must evaluate "correctly".

**Theorem:** If $\vdash e:\tau$ then $e\Downarrow v$ and $\vdash v:\tau$

# Static vs dynamic typing
Languages can have static or dynamic type systems.
## Static typing
Some expressions are not well-typed and thus not well-formed. There are some sensible programs which are not allowed, e.g. `1+true` might evaluate to `2` at runtime, but it is not accepted regardless
## Dynamic typing
All expressions are well formed, so any program can be run. 
In this case type errors arise dynamically, but at the cost of higher overhead for tagging and checking during program execution.

Both of these are extremes, generally a "statically" typed program will still handle some errors dynamically.

A dynamically typed language can also be thought of as a statically typed one with a single type.