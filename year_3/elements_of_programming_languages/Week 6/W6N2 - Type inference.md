Even with a small example, such as `map[A][B](f: A => B, x: List[A]): List[B]` specifying the parameters of polymorphic functions quickly becomes impractical, (`map[int][str]`) and this gets worse if there are complex types involved (`map[list[int]][list[int => str]]`). **Type inference** is the inference of type annotations so that the program can be typechecked for a program missing some or all type information.

# Hindley-Milner type inference
Essentially, an expression is typechecked symbolically, and building up constraints on unknown type variables. If there is a common solution for those constraints then all types are inferred, otherwise there is a type error. Constraints can be solved using *[unification](https://en.wikipedia.org/wiki/Unification_(computer_science))*.

>[!attention]  **For the purposes of this course content from here on is non-examinable**
## Example

Consider $swap$ defined as follows:
$$
\vdash\lambda x:A.(\text{snd }x,\text{fst }x):B
$$
$A,B$ are the currently unknown types of $x$ and $swap$.


A lambda abstraction creates a function and hence, $B=A\rightarrow A_1$ for some $A_1$ such that $x:A\vdash(\text{snd }x,\text{fst }x):A_1$.

A pair constructs a pair type, hence $A_1=A_2\times A_3$ where $x:A\vdash \text{snd }x:A_2$ and $x:A\vdash\text{fst }x):A_3$.

This all can only be the case when $x:A_3\times A_2$, so $A=A_3\times A_2$.

We now have $A=A_3\times A_2$, $A_1=A_2\times A_3$, and so $B=A_3\times A_2\rightarrow A_2\times A_3$.

# Let bound polymorphism
When a function is defined using $\text{let fun}$ or $\text{let rec}$ we infer a type for that function:
$$
swap:A_3\times A_2\rightarrow A_2\times A_3
$$
then abstract over all its free type parameters:
$$
swap:\forall A.\forall B.A\times B\rightarrow B\times A
$$
then, when the function is applied, infer the missing types:
$$
swap(1,\text{"a"})\rightsquigarrow swap[\text{int}][\text{str}](1,\text{"a"})
$$
>[!attention]  **Content from here on is examinable**
# ML-style inference: strengths and weaknesses
Strengths
- Elegant
- Effective
- Requires no type annotation at all

Weaknesses
- Can be difficult to explain errors
- Theoretically can have exponential time complexity
	- In practice it runs efficiently, however it is possible in some languages to deliberately create a program that crashes the typechecker
- Very sensitive to extensions: subtyping and other extensions tend to require giving up some nice properties

# Type inference in Scala
Scala does not employ full HM type inference, but uses many of the same ideas. Type information flows from function arguments to their results, e.g:
```scala
def f[A](x: List[A]): List[(A, A)] = ...
f(List(1, 2, 3))
```
In this case, as `x` is `List[Int]` this is inferred to be the type of `[A]`.
This information is also carried through statement blocks:
```scala
val l = List(1, 2, 3); // l: List[Int] is inferred
val y = f(l); // y: List[(Int, Int)] is inferred
```

Typing information does not flow across arguments in the same argument list:
```scala
def map[A, B](F: A => B, l: List[A]): List[B] = ...
map({x: Int => x + 1}, List(1, 2, 3)) // this does work
map({x => x + 1}, List(1, 2, 3)) // this does *not* work
```

The typing information does flow from earlier argument lists to later ones:
```scala
def map[A, B](l: List[A])(f: A => B): List[B] = ...
map(List(1, 2, 3)){x => x + 1} // this does work
```

>[!note] 
>In a Scala program we will still need to provide some type information, whereas with full HM inference no typing would be required
 
## Strengths and limitations
Scala needs fewer type annotations than Java, but still requires many more than Haskell. The reason for this is due to Scala's integration of polymorphism and subtyping, which is needed for integration with the Java-style object/class system. (Combining polymorphism and subtyping is difficult, and type inference can become undecidable)