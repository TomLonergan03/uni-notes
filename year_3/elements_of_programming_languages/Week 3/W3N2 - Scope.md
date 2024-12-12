In programming we can generally reuse variable names:
```scala
def foo(x: Int) = x + 1
def bar(x: Int) = x * x
```
The occurrences of `x` in `foo` have nothing to do with those in `bar`.

# Definition
The *scope* of a variable name is the collection of program locations in which occurrences of the variable refer to the same thing.

N.b. "refer to the same thing" doesn't necessarily mean that the two variable occurrences evaluate to the same value at run time.

# Binding
```scala
def foo(x: Int) = x + 1
def bar(x: Int) = x * x
```
In this case, the `x: Int` are binding `x` to the scope of the functions. The occurrences of `x` to the right of the `=` are bound.

Essentially, renaming all binding and bound occurrences in a scope consistently (avoiding name clashes) should not change the meaning of the programming

# Let-binding
Let-binding is a basic form of scope:
$$
e::=...|x|\text{let }x=e_1\text{ in }e_2
$$
This is now $L_{Let}$, $L_{If}$ extended with variables and $\text{let}$.
Given the let expression $\text{let }x=e_1\text{ in }e_2$ we say $x$ is bound in $e_2$.

Let binding allows us to use a variable $x$ as an abbreviation for the value of some other expression.
e.g. 
$$
\text{let }x=1+2\text{ in }4\times x\rightsquigarrow\text{let }x=3\text{ in }4\times x\rightsquigarrow4\times 3
$$
## Equivalence up to consistent renaming
We consider expressions equivalent (written $e_1\equiv e_2$) if they have the same binding structure.
We can rename bound names to get equivalent expressions:
$$
\text{let }x=y+z\text{ in }x==w\equiv\text{let }u=y+z\text{ in }u==w
$$
Some renamings can change the binding structure:
$$
\text{let }x=y+z\text{ in }x==w\not\equiv\text{let }w=y+z\text{ in }w==w
$$
Intuition: renaming $x$ to $u$ is alright, as $u$ was not already used, however renaming to $w$ is not alright as $w$ is "in use".

# Free variables
The set of free variables in an expression is defined as:
$$
\begin{align}
FV(n)&=\emptyset)\\
FV(x)&=\{x\}\\
FV(e_1\oplus e_2)&=FV(e_1)\cup FV(e_2)\\
FV(\text{if }e\text{ then }e_1\text{ else }e_2)&=FV(e)\cup FV(e_1)\cup FV(e_2)\\
FV(\text{let }x=e_1\text{ in }e_2)&=FV(e_1)\cup(FV(e_2)-\{x\})
\end{align}
$$
where $X-Y$ is the set of elements of $X$ that are not in $Y$

Examples:
$$
FV(x+y) = \{x,y\}
$$
$$
FV(\text{let }x=y\text{ in }x) = \{y\}
$$
$$
FV(\text{let }x=x+y\text{ in }z) = \{x,y,z\}
$$
# Renaming
We will use the following swapping operation to rename variables:
$$
\begin{align}
x(y\leftrightarrow z)&=
\begin{cases}
	y &\text{if }x=z\\
	z &\text{if }x=y\\
	x &\text{otherwise}
\end{cases}\\
v(y\leftrightarrow z)&=v\\
(e_1\oplus e_2)(y\leftrightarrow z)&=e_1(y\leftrightarrow z)\oplus e_2(y\leftrightarrow z)\\
(\text{if }e\text{ then }e_1\text{ else }e_2)(y\leftrightarrow z)&=\text{if }e(y\leftrightarrow z)\text{ then }e_1(y\leftrightarrow z)\text{ else }e_2(y\leftrightarrow z)\\
(\text{let }x=e_1\text{ in }e_2)(y\leftrightarrow z)&=\text{let }x(y\leftrightarrow z)=e_1(y\leftrightarrow z)\text{ in }e_2(y\leftrightarrow z)
\end{align}
$$
e.g.
$$
(\text{let }x=y\text{ in }x+z)(y\leftrightarrow z)\rightsquigarrow\text{let }z=y\text{ in }z+x
$$
# Alpha conversion
We can now define "consistent renaming"
Suppose $y\notin FV(e_2)$. Then we can rename a let expression as follows:
$$
\text{let }x=e_1\text{ in }e_2\rightsquigarrow\text{let }y=e_1\text{ in }e_2(x\leftrightarrow y)
$$
This is called *alpha-conversion*.

Two expressions are *alpha-equivalent* if we can convert one to the other using alpha-conversions.

E.g.
$$
\begin{align}
&\;\;\;\;\;\;\;\text{let }x=y+z\text{ in }x==w\\
&\rightsquigarrow_\alpha\text{let }u=y+z\text{ in }(x==w)(x\leftrightarrow u)\\
&=\;\;\text{let }u=y+z\text{ in }x(x\leftrightarrow u)==w(x\leftrightarrow u)\\
&=\;\;\text{let }u=y+z\text{ in }(u==w)
\end{align}
$$
These 2 expressions are alpha equivalent as $u\not\in FV(x==w)$.
$$
\text{let }x=y+z\text{ in }x==w \not\rightsquigarrow_\alpha\text{let }w=y+z\text{ in }w==w
$$