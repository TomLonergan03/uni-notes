So far, we have looked at big-step semantics, i.e. how to evaluate the "whole program" (expression) to a "final value". This is a limited subset of programs, as some (e.g. $1+\text{ true}$) simply do not evaluate, while some may never terminate, either accidentally or deliberately (an operating system should run indefinitely, with no provable time point where it terminates without outside input).

In small-step semantics, we have a collection of rules in the style $e \rightarrowtail e'$ which says how to evaluate an expression "one step at a time". 
If $e_0\rightarrowtail...\rightarrowtail e_n$, then we write $e_0\rightarrowtail^*e_n$, also for $n=0$ we have $e_0\rightarrowtail^*e_0$.
It is also the case that $e\rightarrowtail^*e_n$ iff $e\Downarrow v$, but $\rightarrowtail$ provides more detail on how this happens and allows an expression to "go wrong" or get stuck before reaching a value.
# Small-step semantics for [[W1N2 - Abstract syntax#BNF grammars|L_Arith]] 
$$
\begin{gathered}
\frac{e_1\rightarrowtail e'_1}{e_1\oplus e_2\rightarrowtail e'_1\oplus e_2}&
\frac{e_2\rightarrowtail e'_2}{v_1\oplus e_2\rightarrowtail v_1\oplus e'_2}\\
\frac{}{v_1+v_2\rightarrowtail v_1+_\mathbb{N}v_2}&
\frac{}{v_1\times v_2\rightarrowtail v_1\times_\mathbb{N}v_2}
\end{gathered}
$$
Essentially, if the first subexpression $e_1$ can take a step, do so. If the first subexpression is a value and the second can take a step, do so. If both are values, then perform the operation.
E.g.
$$
1+(2\times3)\rightarrowtail1+6\rightarrowtail7
$$
# Small-step semantics for [[W2N2 - Booleans and conditionals|L_If]]
$$
\begin{gathered}
\frac{}{v==v\rightarrowtail\text{true}}&
\frac{v_1\ne v_2}{v==v\rightarrowtail\text{false}}
\end{gathered}
$$
$$
\begin{gathered}
\frac{e\rightarrowtail e'}{\text{if }e\text{ then }e_1\text{ else }e_2\rightarrowtail\text{if }e'\text{ then }e_1\text{ else }e_2}\\
\frac{}{\text{if true then }e_1\text{ else }e_2\rightarrowtail e_1}\\
\frac{}{\text{if false then }e_1\text{ else }e_2\rightarrowtail e_2}
\end{gathered}
$$
We step-wise evaluate $e$ until it is a value, then evaluate the corresponding branch
E.g.
$$
\begin{aligned}
\text{if }1==2\text{ then }3+4\text{ else }4+5&\rightarrowtail
\text{if }1==2\text{ then }3+4\text{ else }4+5\\
&\rightarrowtail \text{if false then }3+4\text{ else }4+5\\
&\rightarrowtail4+5\\
&\rightarrowtail9
\end{aligned}
$$
# Small-step semantics for [[W3N1 - Variables and substitution#Variables|L_Let]]
$$
\begin{gathered}
\frac{e_1\rightarrowtail e'_1}{\text{let }x=e_1\text{ in }e_2\rightarrowtail\text{let }x=e'_1\text{ in }e_2}\\
\frac{}{\text{let }x=v_1\text{ in }e_2\rightarrowtail e_2[v1/x]}
\end{gathered}
$$
If $e_1$ is not yet a value, evaluate it one step, otherwise substitute it and proceed.
E.g.
$$
\begin{aligned}
\text{let }x=1+1\text{ in }x\times x&\rightarrowtail
\text{let }x=2\text{ in }x\times x\\
&\rightarrowtail2\times2\\
&\rightarrowtail4
\end{aligned}
$$
# Small-step semantics for [[W3N4 - Functions#Anonymous functions|L_Lam]]
$$
\begin{gathered}
\frac{e_1\rightarrowtail e'_1}{e_1\;e_2\rightarrowtail e'_1\;e_2}&
\frac{e_2\rightarrowtail e'_2}{v_1\;e_2\rightarrowtail v_1\;e'_2}
\end{gathered}
$$
$$
\frac{}{(\lambda x.e)\;v\rightarrowtail e[v/x]}
$$
E.g.
$$
\begin{aligned}
((\lambda x.\lambda y.x+y)\;1)\;2&\rightarrowtail(\lambda y.1 + y)\;2\\
&\rightarrowtail1+2\\
&\rightarrowtail3
\end{aligned}
$$
# Small-step semantics for [[W3N4 - Functions#Recursive functions|L_Rec]]
$$
\frac{}{(\text{rec }f(x).e)\;v\rightarrowtail e[\text{rec }f(x).e/f,v/x]}
$$
E.g. for a factorial function $fact(x)=\text{if }x==1\text{ then }1\text{ else }x\times fact(x-1)$:
$$
\begin{aligned}
fact(2)&\rightarrowtail\text{if }2==1\text{ then }1\text{ else }2\times fact(2-1)\\
&\rightarrowtail\text{if false then }1\text{ else }2\times fact(2-1)\\
&\rightarrowtail2\times fact(2-1)\\
&\rightarrowtail2\times fact(1)\\
&\rightarrowtail2\times(\text{if }1==1\text{ then }1\text{ else }1\times fact(1-1)\\
&\rightarrowtail2\times(\text{if true then }1\text{ else }1\times fact(1-1))\\
&\rightarrowtail2\times1\\
&\rightarrowtail2
\end{aligned}
$$
