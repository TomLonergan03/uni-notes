# Variables
A variable is a symbol that can 'stand for' a values.
Often written as `x,y,z,...`
Let's extend [[W2N3 - Types|LIf]] with variables to get $L_{Var}$:
$$
\begin{align}
e ::=&n\in\mathbb{N}|e_1+e_2|e_1\times e_2\\
&|b\in\mathbb{B}|e_1==e_2|\text{if }e\text{ then }e_1\text{ else }e_2\\
&|x\in Var
\end{align}
$$
Here, $x$ is shorthand for an arbitrary variable in $Var$, the set of expression variables.
## Schematic syntax
We can represent binary operators using the *schematic syntax* $e_1\oplus e_2$ 
$$
\begin{matrix}
	\frac{e_1\Downarrow v_1\text{ }e_2\Downarrow v_2}{e_1\oplus e_2\Downarrow v_1\oplus_\mathbb{A}v_2} &
	\frac{\vdash e_1:\tau_1\text{ }\vdash e_2:\tau_2\text{ }\oplus:\tau_1\times\tau_2\rightarrow\tau}{\vdash e_1\oplus e_2:\tau}
\end{matrix}
$$
Essentially, this allows us to represent any binary operator's evaluation and typing rules using a single operator with arguments $\tau_1,\tau_2$ and result of type $\tau$.
E.g. $+:\text{int}\times\text{int}\rightarrow\text{int}$ or $==:\tau\times\tau\rightarrow\text{bool}$
# Substitution
If we have a variable in the expression $x+1$ and $x$ is defined as 42 then in every case of $x$ it must be able to be replaced with 42. e.g. $x+1\rightsquigarrow42+1$ 

In $L_{Var}$ we define substitution as follows:
Given $e,x,v$ the substitution of $v$ for $x$ in $e$ is an expression written $e[v/x]$.
$$
\begin{align}
v_0[v/x]&=v_0\\
x[v/x]&=v\\
y[v/x]&=y &(x\ne y)\\
(e_1\oplus e_2)[v/x] &= e_1[v/x]\oplus e_2[v/x]\\
(\text{if }e\text{ then }e_1\text{ else }e_2)[v/x]&=\text{if }e[v/x]\text{ then }e_1[v/x]\text{ else }e_2[v/x]
\end{align}
$$