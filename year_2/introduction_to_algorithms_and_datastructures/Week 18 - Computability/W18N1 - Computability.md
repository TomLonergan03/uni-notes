The **Church-Turing (CT) computable functions** are generally accepted as coinciding with the 'algorithmically computable' functions i.e. functions that can be computed using a series of instructions which can be defined as a series of steps and ignoring time and space limitations.

There are many ways to reach the same class of CT-computable functions:
- $\lambda$ calculus
- Turing machines
- Register machines

# Register machines
To use the example of register machines, a register machine has a fixed, finite set of registers ($A,B,...,I$)each capable of storing an arbitrary natural number.
We can build a machine by joining trivial components
![[w18n1RegisterMachineComponents.png]]
The example machine adds $B$ to $A$, losing the value of $B$ in the process.

To use a register to compute a function, e.g. $\mathbb{N}\times\mathbb{N}\rightharpoonup\mathbb{N}$,  we can supply inputs in registers $A$ and $B$, and read the output from $A$.

# Church-Turing thesis
The **Church-Turing thesis** claims that the class of CT computable functions coincides with the class of all algorithmically computable functions.
This is not a strict mathematical proof as algorithmically computable functions is only an informal class.

## Why accept the CT thesis
1. Nobody has ever come up with a 'mechanical' algorithm which computes something outside this class (no algorithm exists that cannot be computed using $\lambda$ calculus/register machines/Turing machines)
2. There have been many attempts to define a 'computable' function that all converge on the same class
3. Turing's argument was to consider what a human calculator could do in principle with:
	1. finitely many distinguishable mind states
	2. unlimited paper, but finitely many distinguishable symbols
	3. finitely many 'fingers on the page'
Regardless of which argument is used, the CT thesis is solidly established and is safe to build on.

# The universal machine
We can produce a way to encode a number of register values within a single value, for example A=45, B=5, C=132 we could choose to encode this as 552403001, with the 1s first, then 10s, then 100s, and so on (this is just an example encoding) and a way to encode a specific machine in a value.
Then, with an appropriate encoding, it would be possible to recover the machine and input register values using an algorithm.
Therefore, it is possible to build a register machine to do it, and also to simulate what that machine would do.
This is the **universal machine** $U$.

![[w18n1UniversalMachine.png]]

As the universal machine is just a complex register machine, it is capable of simulating any other register machine, or even itself.

Turing's insight that all machines could be simulated by a single machine led to the concept of building one computer that can be programmed to complete many computing tasks. This is the origin of the modern general-purpose, programmable computer.