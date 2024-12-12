# Type abbreviation
When using [[W5N1 - Records, named variants, and pattern matching|records or variants]] it quickly becomes painful to write "$\langle x:\text{int},y:\text{str}\rangle$" over and over.
A **type abbreviation** gives a name for a type:
$$
\text{type }T=\tau
$$
a type abbreviation $T$ is treated the same as its expansion $\tau$, similar to [[W3N2 - Scope#Let-binding|let-bound variables]]

e.g.
$$
\begin{aligned}
&\text{type } Point = \langle x:\text{dbl},y:\text{dbl}\rangle\\
&\text{type } Point3d = \langle x:\text{dbl},y:\text{dbl},z:\text{dbl}\rangle\\
&\text{type } Color = \langle r:\text{int},g:\text{int},b:\text{int}\rangle\\
&\text{type } ColoredPoint = \langle x:\text{dbl},y:\text{dbl},c:Color\rangle\\
\end{aligned}
$$

# Type definition
A type abbreviation is equivalent to the type it abbreviates, e.g. using the above definitions, if a function takes a $Point$, it will also accept a $\langle x:\text{dbl},y:\text{dbl}\rangle$. A **type definition** creates a new type which does not typecheck to the same result.
$$
\text{deftype }T=\tau
$$

Type abbreviations are not usually allowed to be recursive, while type definitions often are:
$$
\text{type }IntList=[Nil:\text{unit},Cons:\text{int}\times IntList]
$$
# Type definitions vs abbreviations in practice
Haskell has type abbreviations with `type`, while new types are defined using `data` or `newtype`.
Java has no notation for type abbreviations, and the only way to define a new type is using a `class` or `interface`.
Scala introduces abbreviations using `type`, while `class`, `object`, and `trait` define new types.