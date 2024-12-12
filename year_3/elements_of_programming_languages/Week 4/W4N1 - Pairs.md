The simplest way to combine data structures is pairing:
$$
\begin{matrix}
(1,2)&(\text{true},\text{false})&(1,(\text{true},\lambda x:int.x + 2))
\end{matrix}
$$
If we have a pair we can extract one of the components:
$$
\begin{matrix}
\text{fst }(1,2)\rightsquigarrow1&
\text{snd }(\text{true},\text{false})\rightsquigarrow\text{false}
\end{matrix}
$$
$$
\text{snd } (1,(\text{true}, \lambda x:\text{int}.x+2))\rightsquigarrow(\text{true}, \lambda x:\text{int}.x+2)
$$
We can often also pattern match against a pair, to extract both components at once:
$$
\text{let pair }(x,y)=(1,2)\text{ in }(y,x)\rightsquigarrow(2,1)
$$
## In various languages
| Haskell        | Scala          | Java             | Python     |
| -------------- | -------------- | ---------------- | ---------- |
| `(1, 2)`       | `(1, 2)`       | `new Pair(1, 2)` | `(1, 2)`   |
| `fst e`        | `e._1`         | `e.getFirst()`   | `e[0]`     |
| `snd e`        | `e._2`         | `e.getSecond()`  | `e[1]`     |
| `let (x, y) =` | `val (x, y) =` | N/A              | `(x, y) =` |

## Syntax and semantics
$$
\begin{align}
e &::= ...|(e_1,e_2)|\text{fst }e|\text{snd }e\\
&\;\text{let pair }(x,y)=e_1\text{ in }e_2\\
v &::= ...|(v_1,v_2)
\end{align}
$$
$$
\begin{matrix}
\frac{e_1\Downarrow v_1\\;\;\;\;e_2\Downarrow v_2}{(e_1,e_2)\Downarrow(v_1,v_2)}&
\frac{e\Downarrow(v_1,v_2)}{\text{fst }e\Downarrow v_1}&
\frac{e\Downarrow(v_1,v_2)}{\text{snd }e\Downarrow v_2}
\end{matrix}
$$
$$
\frac{e_1\Downarrow(v_1,v_2)\\;\;\;\;e_2[v_1/x,v_2/y]\Downarrow v}{\text{let pair }(x,y)=e_1\text{ in }e_2\Downarrow v}
$$
## Types
$$
\tau::=...|\tau_1\times\tau_2
$$
$$
\frac{\Gamma\vdash e_1:\tau_1\;\;\;\;\Gamma\vdash e_2:\tau_2}{\Gamma\vdash(e_1,e_2):\tau_1\times\tau_2}
$$
$$
\begin{matrix}
\frac{\Gamma\vdash e:\tau_1\times\tau_2}{\Gamma\vdash\text{fst } e:\tau_1}&
\frac{\Gamma\vdash e:\tau_1\times\tau_2}{\Gamma\vdash\text{snd } e:\tau_2}&
\end{matrix}
$$
$$
\frac{\Gamma\vdash e_1:\tau_1\times\tau_2\;\;\;\;\Gamma,x:\tau_1,y:\tau_2\vdash e_2:\tau}{\Gamma\vdash\text{let pair }(x,y)=e_1\text{ in }e_2:\tau}
$$
## Let vs fst and snd
The $\text{fst}$ and $\text{snd}$ operations are definable in terms of $\text{let pair}$:
$$
\begin{aligned}
\text{fst }e\;&\Longleftrightarrow\;\text{let pair }(x,y)=e\text{ in }x\\
\text{snd }e\;&\Longleftrightarrow\;\text{let pair }(x,y)=e\text{ in }y
\end{aligned}
$$
Likewise, $\text{let pair}$ can be defined in terms of $\text{let, fst, snd}$:
$$
\begin{aligned}
\text{let pair }&(x,y)=e_1\text{ in }e_2\\
\Longleftrightarrow&\text{let }p=e_1\text{ in }e_2[\text{fst }p/x,\text{ snd }p/y]
\end{aligned}
$$

Typically, we use $\text{fst}$ and $\text{snd}$ and treat $\text{let pair}$ as syntactic sugar.

# n-tuples
n-tuples are equally easy to add, but can be defined using pairs so there is generally no need for them.

# Unit type (0-tuples)
We can also add a type of 0-tuples, which are a data structure containing no data. This is called the unit type.
$$
\begin{aligned}
e&::=...|()\\
v&::=...|()\\
\tau&::=...|\text{unit}\\
\end{aligned}
$$
$$
\begin{matrix}
\frac{}{()\Downarrow()}&
\frac{}{\Gamma\vdash():\text{ unit}}
\end{matrix}
$$
This may seem pointless, however it is actually useful e.g. `void` in C/Java, or `()` in Haskell and Scala