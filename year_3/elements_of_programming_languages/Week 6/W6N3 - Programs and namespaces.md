We will now focus on programs "in the large", such as whole programs, packages, modules, and interfaces. We will mostly work "by example" in Scala, as formalising modules and interfaces involves a lot of bureaucracy.

# Programs
## What is a program?
In $L_{Poly}$, a program is an expression, with any functions defined in it are local to the expression, this would look something like:
$$
\begin{aligned}
&\text{let fun }f(x:\tau)=e_1\text{ in}\\
&\text{let fun }g(x:\tau_1)=e_2\text{ in}\\
&...\\
&e
\end{aligned}
$$
where we have a list of definitions, then finally an expression to evaluate with all that information.
## Declarations
Most languages support *declarations*
$$
\begin{aligned}
Decl\ni d\;&::=\;\text{let }x=e;\:|\:\text{let fun }f(y:\tau)=e;\\
&::=\:\text{let rec }f(y:\tau):\tau'=e;\\
&::=\:\text{type }T=\tau;\:|\:\text{deftype }T=\tau;
\end{aligned}
$$

Here, a program is a sequence of declarations.
In some languages (e.g. Haskell, Scala), the order of declarations are unimportant.
In some other languages , only certain "top level" declarations are allowed (e.g. Java only allows classes and interfaces to be defined at the top level)
## Entry points
The **entry point** of a program is the place where execution starts when the program is run.
This can be specified in different ways:
- **Executables** often have a particular function that is always called first
  E.g. in Java
  ```java
  public static void main(String[] args) {...}
  ```
- **Scripting** often has the entry point is the first line of the program, and expressions or statements are run in order
- **Web applications** often have entry points being functions, e.g. `doGet` and `doPost` in Java's `Servlet`
- **Reactive:** callbacks are provided for one or more events, e.g. JavaScript handlers for mouse actions

# Namespaces and packages
As a program gets larger, eventually people will want to use the same name for different things. **Namespaces** allow a function to be compartmentalised.

E.g. in Java:
```java
// com/widget/round/Widget.java
package com.widget.round
class Widget {
...
}

// com/widget/square/Widget.java
package com.widget.square
class Widget {
...
}
```
Here we can reuse the class name `Widget` for 2 different things, and distinguish between whether we want to use `com.widget.square.Widget` or `com.widget.round.Widget`. (Package names track the directory in Java, but this isn't true for most other languages with namespaces/packages)

## Importing
Given a namespace we can import it:
```java
import com.widget.round.Widget
```
which brings a single name into the current scope, or:
```java
import com.widget.round.*
```
which brings all names in that namespace into the current scope