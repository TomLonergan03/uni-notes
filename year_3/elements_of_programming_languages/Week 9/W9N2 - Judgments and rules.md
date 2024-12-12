A judgement is a relation among one or more ASTs, examples include $e\Downarrow v$, $\Gamma\vdash e:\tau$, $e\rightarrowtail e'$.

We have been defining judgements using rules of the form:
$$
\begin{gathered}
\frac{}{Q}&\frac{P_1\;\;\;\;...\;\;\;\;P_n}{Q}
\end{gathered}
$$
where $P_1,...,P_n$ and $Q$ are judgements.

A rule of the form:
$$
\frac{}{Q}
$$
is called an axiom. It says that $Q$ is always derivable.

A rule of the form:
$$
\frac{P_1\;\;\;\;...\;\;\;\;P_n}{Q}
$$
says that $Q$ is derivable if $P_1,...,P_n$ are derivable.

Symbols like $e,v,\tau$ in rules stand for arbitrary expressions, values, or types respectively.

# Rule induction
>[!summary] Induction on derivations of $e\Downarrow v$
>Suppose $P(\textendash,\textendash)$ is a predicate over pairs of expressions and values. If:
>- $P(v,v)$ holds for all values $v$
>- If $P(e_1,v_1)$ and $P(e_2,v_2)$ then $P(e_1+e_2,v_1+_\mathbb{N}v_2)$
>- If $P(e_1,v_1)$ and $P(e_2,v_2)$ then $P(e_1\times e_2,v_1\times_\mathbb{N}v_2)$
>then $e\:\Downarrow\:v$ implies $P(e,v)$

Rule induction can be derived from mathematical induction on the size or height of the derivation tree, much like [[W1N2 - Abstract syntax#Structural induction|structural induction]], though we won't formally prove this.

Example: $e\Downarrow v$ implies $e\rightarrowtail^*v$

**Base case**: If the derivation is of the form
$$
\frac{}{n\Downarrow n}
$$
for some number $n$ then $e=n$ is already a value $v=n$, so no steps are needed to evaluate it, i.e. $n\rightarrowtail^*n$ in zero steps.

**Inductive case**: If the derivation is of the form
$$
\frac{e_1\Downarrow v_1\;\;\;\;e_2\Downarrow v_2}{e_1+e_2\Downarrow v_1+v_2}
$$
then by induction, we know $e_1\rightarrowtail^*v_1$ and $e_2\rightarrowtail^*v_2$. Using the small step rules, we can then show
$$
e_1+e_2\rightarrowtail^*v_1+e_2\rightarrowtail^*v_1+v_2\rightarrowtail v_1+_\mathbb{N}v_2
$$
The case for $\times$ is similar.

