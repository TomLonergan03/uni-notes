If we have a function which computes the distance to the origin:
$$
dist=\lambda p:Point.sqrt((p.x)^2+(p.y)^2)
$$
Only the $x$ and $y$ fields are needed, so we'd like to be able to use this on *[[W5N2 - Type abbreviations and definitions#Type abbreviation|ColoredPoints]]* as well. This wouldn't typecheck, even though it would evaluate correctly.

We can introduce as **subtyping** relationship between *Point* and *ColoredPoint* to allow for this using the **Liskov Substitution Principle**:
If $S$ is a subtype of $T$, then objects of type $T$ may be replaced with objects of type $S$ without altering any of the desirable properties of the program.

If we use $\tau<:\tau'$ to mean "$\tau$ is a subtype or $\tau'$" then we can create this *subsumption* rule:
$$
\frac{\Gamma\vdash e:\tau_1\;\;\;\;\tau_1<:\tau_2}{\Gamma\vdash e:\tau_2}
$$
This says: if $e$ has type $\tau_1$ and $\tau_1<:\tau_2$, then we can proceed by pretending it has type $\tau_2$.

# Record subtyping
## Width subtyping
The subtype has the same fields of the supertype (with identical types), and may have additional extra fields at the end:
$$
\frac{}{\langle I_1:\tau_1,...,I_n:\tau_n,...,I_{n+k}:\tau_{n+k}\rangle<:\langle I_1:\tau_1,...,I_n:\tau_n\rangle}
$$
## Depth subtyping
The subtypes fields are pointwise subtypes of the supertype:
$$
\frac{\tau_1<:\tau'_1\;\;\;\;...\;\;\;\;\tau_n<:\tau'_n}{\langle I_1:\tau_1,...,I_n:\tau_n\rangle<:\langle I_1:\tau'_1,...,I_n:\tau'_n\rangle}
$$

These rules can also be combined, and it is possible (but difficult to implement) field reordering also.

# Subtypes for pairs and variants
For [[W4N1 - Pairs|pairs]], subtyping is componentwise:
$$
\frac{\tau_1<:\tau'_1\;\;\;\;\tau_2<:\tau'_2}{\tau_1\times\tau_2<:\tau'_1\times\tau'_2}
$$
and similar for [[W4N2 - Variant types|binary variants]]:
$$
\frac{\tau_1<:\tau'_1\;\;\;\;\tau_2<:\tau'_2}{\tau_1+\tau_2<:\tau'_1+\tau'_2}
$$
[[W5N1 - Records, named variants, and pattern matching#Named variants|Named variants]] can also have additional subtyping rules, but this is rare.

# Subtyping for functions
The result type of a function can be found using subsumption as usual (*covariant*), however for the argument type the direction of subtyping is flipped (*contravariant*):
$$
\frac{\tau'_1<:\tau_1\;\;\;\;\tau_2<:\tau'_2}{\tau_1\rightarrow\tau_2<:\tau'_1\rightarrow\tau'_2}
$$
# Top and bottom types
## Top
$any$: a type that is a supertype of all types
- This type describes the common interface of all its subtypes
- In Scala this is called `Any`

# Bottom
$empty$: a type that is a subtype of all types
- Usually, such a type is empty: there cannot be any values of this type
- In Scala, this is called `Nothing`. For all Scala types $\tau$ we have $Nothing<:\tau<:Any$.

# Subtyping rules
$$
\begin{matrix}
\frac{}{\text{empty}<:\tau}&
\frac{}{\tau<:\text{any}}&
\frac{}{\tau<:\tau}&
\frac{\tau_1<:\tau_2\;\;\;\;\tau_2<:\tau_3}{\tau_1<:\tau_3}
\end{matrix}
$$
$$
\begin{matrix}
\frac{\tau_1<:\tau'_1\;\;\;\;\tau_2<:\tau'_2}{\tau_1\times\tau_2<:\tau'_1\times\tau'_2}&
\frac{\tau_1<:\tau'_1\;\;\;\;\tau_2<:\tau'_2}{\tau_1+\tau_2<:\tau'_1+\tau'_2}
\end{matrix}
$$
$$
\frac{\tau'_1<:\tau_1\;\;\;\;\tau_2<:\tau'_2}{\tau_1\rightarrow\tau_2<:\tau'_1\rightarrow\tau'_2}
$$
