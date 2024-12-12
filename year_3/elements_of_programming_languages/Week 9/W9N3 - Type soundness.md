The central property of a type system is *soundness*. Roughly, soundness means "well-typed programs don't go wrong" \[Milner].

What does "go wrong" mean?
For large-step semantics it's hard to say, but for small step semantics it means getting "stuck" with an expression $e$ which is not a value but there are no next steps for.
We can then show something like:
>[!summary] Theorem (Value Soundness)
>If $\vdash e:\tau$ and $e\rightarrowtail^*v$ then $\vdash v:\tau$

i.e. if an expression evaluates to a value, then that value has the correct type

>[!summary] Lemma (Progress)
>If $\vdash e:\tau$ then $e$ is not stuck: that is, either $e$ is a value, or for some $e'$ we have $e\rightarrowtail e'$

>[!summary] Lemma (Preservation)
>If $\vdash e:\tau$ and $e\rightarrowtail e'$ then $\vdash e':\tau$

We can combine these to show:
>[!summary] Theorem (Soundness)
>If $\vdash e:\tau$ then $e$ is not stuck and if $e\rightarrowtail^*e'$ then $\vdash e':\tau$

Progress is proved by induction on $\vdash e:\tau$ derivations.

## Progress for $+$:
$$
\frac{\vdash e_1:\text{int}\;\;\;\;\vdash e_2:\text{int}}{\vdash e_1+e_2:\text{int}}
$$
If the derivation is of the above form, then by induction $e_1$ is either a value or can take a step, and likewise for $e_2$. There are three cases:
- If $e_1\rightarrowtail e'_1$ then $e_1+e_2\rightarrowtail e'_1+e_2$
- If $e_1$ is a value $v_1$ and $e_2\rightarrow e'_2$, then $v_1+e_2\rightarrowtail v_1+e'_2$
- If both $e_1$ and $e_2$ are values then they must both be numbers $n_1,n_2\in\mathbb{N}$, so $e_1+e_2\rightarrow n_1+_\mathbb{N}n_2$

## Progress for $\text{If}$:
$$
\frac{\vdash e:\text{bool}\;\;\;\;\vdash e_1:\tau\;\;\;\;\vdash e_2:\tau}{\vdash\text{if }e\text{ then }e_1\text{ else }e_2:\tau}
$$
If the derivation is of the above form, then by induction, either $e$ is a value or can take a step. There are two cases:
- If $e\rightarrowtail e'$ then $\text{if }e\text{ then }e_1\text{ else }e_2\rightarrowtail\text{if }e'\text{ then }e_1\text{ else }e_2$
- If $e$ is a value, then it must be either $\text{true}$ or $\text{false}$. In the first case, $\text{if true then }e_1\text{ else }e_2\rightarrowtail e_1$, otherwise $\text{if false then }e_1\text{ else }e_2\rightarrowtail e_2$

## Preservation for $+$
$$
\frac{\vdash e_1:\text{int}\;\;\;\;\vdash e_2:\text{int}}{\vdash e_1+e_2:\text{int}}
$$
If the derivation is of the above form, there are three cases:
- If $e_i=v_i$ and $v_1+v_2\rightarrowtail v_1+_\mathbb{N}v_2$ then obviously $\vdash v_1+_\mathbb{N}v_2:\text{int}$
- If $e_1+e_2\rightarrowtail e'_1+e_2$ where $e_1\rightarrowtail e'_1$, then since $\vdash e_1:\text{int}$, we have $\vdash e'_1:\text{int}$, so $\vdash e'_1+e_2:\text{int}$ also
- The case where $e_1=v_1$ and $v_1+e_2\rightarrowtail v_1+e'_2$ is similar

## Preservation for $\text{if}$
$$
\frac{\vdash e:\text{bool}\;\;\;\;\vdash e_1:\tau\;\;\;\;\vdash e_2:\tau}{\vdash\text{if }e\text{ then }e_1\text{ else }e_2:\tau}
$$
If the derivation is of the above form, then there are three cases:
- If $\text{if }e\text{ then }e_1\text{ else }e_2\rightarrowtail\text{if }e'\text{ then }e_1\text{ else }e_2$, then by induction we can show that $\vdash e':\text{bool}$ and $\vdash \text{if }e'\text{ then }e_1\text{ else }e_2:\tau$
- If $e=\text{true}$ then $\text{if true then }e_1\text{ else }e_2\rightarrowtail e_1$, so we already know $\vdash e_1:\tau$
- The case for $\text{if false then }e_1\text{ else }e_2\rightarrowtail e_2$ is similar
