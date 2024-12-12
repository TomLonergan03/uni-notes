# Values
We will use the expression defined [[W1N2 - Abstract syntax#BNF grammars|here]].
In this case, there are some expressions, like 1 or 2 or 3, which are special as they have no remaining "computation" to do. These expressions are called **values**.
We can define a [[W1N2 - Abstract syntax#BNF grammars|BNF grammar]] rule for values:
$$Value\ni v::=n\in\mathbb{N}$$
# Evaluation, informally
Given an expression $e$, what is its value?
- If $e=n$, then it is already a value
- If $e=e_1+e_2$, then evaluate $e_1$ to $v_1$ and $e_2$ to $v_2$. Then add  $v_1$ and $v_2$ together, to get the value of $e$.
- If $e=e_1\times e_2$, then evaluate $e_1$ to $v_1$ and $e_2$ to $v_2$. Then multiply  $v_1$ and $v_2$ together, to get the value of $e$.

## Evaluation in Scala
```scala
def eval(e: Expr): Int = e match {
	case Num(n) => n
	case Plus(e1, e2) => eval(e1) + eval(e2)
	case Times(e1, e2) => eval(e1) * eval(e2)
}
```
(`Expr` is defined [[W1N2 - Abstract syntax#Scala|here]])

# Evaluation, formally
To specify and reason about evaluation, we use a *evaluation judgement*.

**Definition:** Given an expression $e$ and value $v$, we say $v$ is the value of $e$ if evaluating $e$ results in $v$ and we write $e\Downarrow v$ to indicate this.

A judgement is a relation between [[W1N2 - Abstract syntax#Abstract syntax trees|ASTs]] (in this case ASTs evaluate to an AST containing a single value).

E.g.
$1+2\Downarrow3$
$1+2\times3\Downarrow7$
$(1+2)\times3\Downarrow9$
## Evaluation of values
A value is already evaluated, so for any $v$ we have $v\Downarrow v$. This always holds so we can express it as $\overline{v\Downarrow v}$, i.e. there are no preconditions to evaluate $v$.

## Evaluation of addition
If we want to evaluate $e_1+e_2$, we must first evaluate $e_1\Downarrow v_1$ and $e_2\Downarrow v_2$. We can express this as:
$$
\frac{e_1\Downarrow v_1\text{ }e_2\Downarrow v_2}{e_1+e_2\Downarrow v_1+_\mathbb{N}v_2}
$$
n.b. we use $+_\mathbb{N}$ for the mathematical function to add 2 numbers to differentiate from the AST $v_1+v_2$.

To evaluate $e_1\times e_2$ we have a similar expression.

We can use these rules to derive evaluation judgements for complex expressions:
![[w2n1DerivedJudgement.png]]
These are derivation trees, showing how we can derive a conclusion from axioms. A left node must match a rule with no preconditions, all other nodes must match rules with preconditions.

# Totality and structural induction
Given a property $P$ of expressions, if:
- $P(n)$ holds for every number $n\in\mathbb{N}$
- For any expressions $e_1,e_2$, if $P(e_1)$ and $P(e_2)$ hold then $P(e_1+e_2)$ also holds
- For any expressions $e_1,e_2$, if $P(e_1)$ and $P(e_2)$ hold then $P(e_1\times e_2)$ also holds
Then $P(e)$ holds for all expressions $e$.

## Example proof

**Theorem:** If $e$ is an expression conforming to $L_{Arith}$, then there exists $v\in\mathbb{N}$ such that $e\Downarrow v$ holds.

**Proof:**
- Base case:
	- If $e=n$ then $e$ is already a value. Take $v=n$, then we can derive $$\overline{e\Downarrow v}$$
- Induction case 1:
	- If $e=e_1+e_2$ then suppose $e_1\Downarrow v_1$ and $e_2\Downarrow v_2$ for some $v_1,v_2$. Then we can use the rule: $$\frac{e_1\Downarrow v_1,e_2\Downarrow v_2}{e_1+e_2\Downarrow v_1+_{\mathbb{N}}v_2}$$
- Induction case 2:
	- Similarly for $e=e_1\times e_2$ (this is just for brevity, you should prove all cases in full)
Therefore every expression $e$ conforming to $L_{Arith}$ is evaluatable, so $L_{Arith}$ has totality.

# Uniqueness
Uniqueness states that every evaluation of the same expression produces the same value.
## Theorem
If $e\Downarrow v$ and $e\Downarrow v'$ then $v=v'$.
## Proof
- Base case:
	- If $e=n$ then since $n\Downarrow v$ and $n\Downarrow v'$, the only way these judgements can be derived is for $v,v'$ to both equal $n$.
- Induction case 1:
	- If $e=e_1+e_2$ then suppose $e_1\Downarrow v_1$ and $e_2\Downarrow v_2$ for some $v_1,v_2$. Then the derivations must be of the form: 
	  $$
	 \begin{matrix}
		 \frac{e_1\Downarrow v_1,e_2\Downarrow v_2}{e_1+e_2\Downarrow v_1+_{\mathbb{N}}v_2} &
		 \frac{e_1\Downarrow v'_1,e_2\Downarrow v'_2}{e_1+e_2\Downarrow v'_1+_{\mathbb{N}}v'_2} &
	 \end{matrix}
	 $$
	 - By induction, $e_1\Downarrow v_1$ and $e_1\Downarrow v'_1$ implies $v_1=v'_1$, and similarly for $e_2$. Therefore $v_1+_{\mathbb{N}}v_2=v'_1+_{\mathbb{N}}v'_2$
 - Induction case 2:
	 - Similarly for $e=e_1\times e_2$
Therefore, $L_{Arith}$ is unique