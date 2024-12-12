![[w7n3rippleCarryAdder.png]]
A ripple carry adder is a very straightforward way to construct $N$-bit adders, which have good logical complexity ($O(N)$), but bad logic delay (also $O(N)$). The critical path is the one from $C_0$ to $C_4$, as each adder needs the result of the previous adder for its carry in.
# Carry generation and propagation
There are two sources of carry bits, generation (where bits $A_n$ and $B_n$ are both $1$) and propagation (where one of $A_n$ and $B_n$ are $1$ and $C_n$ is also $1$). This gives us 
$$
\begin{matrix}
	\text{Carry generate} & G_i=A_iB_i\\
	\text{Carry propagate} & P_i=A_i\oplus B_i\\
	\text{Sum} & S_i=P_i\oplus C_i\\
	\text{Carry} & C_{i+1}=G_i+P_iC_i
\end{matrix}
$$
Carry can then be calculted for each term:
$$
\begin{aligned}
C_3&=G_2+P_2C_2\\
C_2&=G_1+P_1C_1\\
C_1&=G_0+P_0C_0
\end{aligned}
$$
which expands to
$$
\begin{aligned}
C_3&=G_2+P_2(G_1+P_1(G_0+P_0C_0))\\
C_2&=G_1+P_1(G_0+P_0C_0)\\
C_1&=G_0+P_0C_0
\end{aligned}
$$
which flattens to
$$
\begin{aligned}
C_3&=G_2+P_2G_1+P_2P_1G_0+P_2P_1P_0C_0\\
C_2&=G_1+P_1G_0+P_1P_0C_0\\
C_1&=G_0+P_0C_0
\end{aligned}
$$
# Parallel prefix addition
We can then make "group propagate" signals $P_{i,j}=P_{i,k}P_{k,j}$ where $i\ge k\ge j$ , and "group generate"  signals $G_{i,j}=G_{i,k}+P_{i,k}G_{k,j}$. These can be partitioned at any point, e.g. $G_{6,0}=G_{6,4}+P_{6,4}G_{4,0}$

We then define 3 operators:
![[w7n3parallelPrefixOperators.png]]
with the fundamental carry operator $\varphi$  computing $G,P$ in terms of left and right subgroups.

These operators are combined into a tree structure:
![[w7n3parallelPrefixAddition.png]]
which is the overall adder, but now the delay for each carry path is $\log_2(N)$, as each signal now only needs to pass through 1 red box and at most 3 yellow circles.