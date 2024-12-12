# Concrete vs Abstract syntax
- **Concrete syntax:** the actual syntax of a programming language
	- Specified using context-free grammars
	- Used in the compiler/interpreter front-end, to interpret strings as programs
- **Abstract syntax:** the "essential" constructs of a programming language
	- Specify using *Backus Naur Form* (BNF) grammars
	- Used in specifications and implementations to describe the *abstract syntax trees* of a language

# Context-free grammar
A context-free grammar gives a concrete syntax for expressions. E.g.
$$
\begin{align}
&E\rightarrow E\text{ PLUS } F\\
&F\rightarrow F\text{ TIMES } F | \text{ NUM } | \text{ LPAREN } E \text{ RPAREN}
\end{align}
$$
This does care about precedence, parentheses, etc.

# BNF grammars
BNF grammars give abstract syntax for expressions. E.g.
$$
Expr \ni e ::= e_1+e_2 | e_1\times e_2|n\in\mathbb{N}
$$
This says: there are 3 kinds of expressions:
- Additions $e_1+e_2$ where 2 expressions are combined with the $+$ operator
- Multiplications $e_1\times e_2$ where 2 expressions are combined with the $\times$ operator
- Numbers $n\in\mathbb{N}$

## BNF conventions
- We will use BNF-style rules to define abstract syntax trees, and assume concrete syntax issues such as precedence, parentheses, whitespace, etc. are handled elsewhere.
- The subscripts on occurrences of $e$ on the RHS don't affect the meaning
- We will freely use parentheses in abstract syntax notation to disambiguate
	- e.g. $(1+2)\times3$ vs $1+(2\times3)$

# Abstract syntax trees
A BNF grammar defines a collection of abstract syntax trees (ASTs), e.g.
![[w1n2AbstractSyntaxTrees.png]]

These can be programmatically represented as trees, or in other ways.

# Implementations of ASTs
### Java
```Java
abstract class Expr {
	abstract public int size();
}

class Num extends Expr {
	public int n;
	Num(int _n) {
		n = _n;
	}
	public int size() { return 1; }
}

class Plus extends Expr {
	public Expr e1;
	public Expr e2;
	Plus(Expr _e1, Expr _e2) {
		e1 = _e1;
		e2 = _e2;
	}
	public int size() {
		return e1.size() + e2.size() + 1;
	}
}

class Times extends Expr {
	// similar
}
```
### Python
```python
class Expr:
	pass

class Num(Expr):
	def __init__(self, n):
		self.n = n

	def size(self):
		return 1

class Plus(Expr):
	def __init__(self, e1, e2):
		self.e1 = e1
		self.e2 = e2

	def size(self):
		return self.e1.size() + self.e2.size() + 1

class Times(Expr): 
	# similar
```

### Haskell
In Haskell, ASTs are defined as datatypes
```haskell
data Expr = Num Integer
		  | Plus Expr Expr
		  | Times Expr Expr

size :: Expr -> Integer
size (Num n) = 1
size (Plus e1 e2) = (size e1) + (size e2) + 1
size (Times e1 e2) = (size e1) + (size e2) + 1
```

### Scala
In Scala, ASTs can be defined using *case classes*:
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
```

## Creating ASTs
### Java
```Java
new Plus(new Num(2), new Num(2));
```
### Python
```python
Plus(Num(2), Num(2))
```
### Haskell
```haskell
Plus(Num(2), Num(2))
```
### Scala
("new" is optional for case classes)
```scala
new Plus(new Num(2), new Num(2))
// or
Plus(Num(2), Num(2))
```

# Structural induction
The three most important reasoning techniques for programming languages are:
- Mathematical induction
	- Induction over $\mathbb{N}$
- Structural induction
	- Induction over ASTs
- Rule induction
	- Induction over derivations

## Mathematical induction
Given a property $P$ of natural numbers, if:
- $P(0)$ holds
- for any $n\in \mathbb{N}$, if $P(n)$ holds then $P(n+1)$ also holds
Then $P(n)$ holds for all $n\in\mathbb{N}$.

## Induction over expressions
Given a property $P$ of expressions, if:
- $P(n)$ holds for every number $n\in\mathbb{N}$
- for any expression $e_1,e_2$, if $P(e_1)$ and $P(e_2)$ holds then $P(e_1+e_2)$ also holds
- for any expression $e_1,e_2$, if $P(e_1)$ and $P(e_2)$ holds then $P(e_1\times e_2)$ also holds
Then $P(e)$ holds for all expressions $e$.

Note that here induction is performed over ASTs, not numbers

### Proof using expression induction principle
Given an expression:
$$
\begin{align}
size(n)&=1\\
size(e_1+e_2)&=size(e_1)+size(e_2)+1\\
size(e_1\times e_2)&=size(e_1)+size(e_2)+1
\end{align}
$$
Given $P(-)$ satisfying the assumptions of expression induction, we need to use induction over $\mathbb{N}$ to show $P(e)$ holds for any $e$. We will use $\mathbb{N}$-induction to prove $Q(n)$ for any $n$ where:
$$
Q(n)=\text{ for all $e$ with $size(e)<n$ we have } P(e)
$$
Since any expression $e$ has a finite size, $P(e)$ holds for any expression because $Q(size(e)+1)$ holds and implies $P(e)$.
#### Proof
We prove that $Q(n)$ holds for all $n$ by induction on $n$:
- The base case $n=0$ is vacuous
- For $n+1$, we assume $Q(n)$ holds and consider any $e$ with $size(e)<n+1$. Then there are 3 cases:
	- if $e=m\in\mathbb{N}$, then $P(e)$ holds by part 1 of the expression induction principle
	- if $e=e_1+e_2$ then $size(e_1)<size(e)\leq n$ and $size(e_2)<size(e)\leq n$. So, by induction, $P(e_1)$ and $P(e_2)$ hold, and by part 2 of expression induction principle $P(e)$ holds.
	- if $e=e_1\times e_2$, the same reasoning applies

(essentially, $Q(n)$ holds as any $Q(n)$ can be broken down into a combination of instances of $Q(m),m<n$, which we know to be true)
