# What is a compiler
In its most basic form, a compiler translates a program from one programming language to another. Generally, the target language is of a "lower" level than the source language. A compiler determines whether a program is syntactically correct and generates code that is semantically identical to the source.
# Two pass compilers
A two-pass compiler has a **frontend**, which translates the source language into an **intermediate representation (IR)**, and a **backend**, which translates the IR into the target language (usually machine code). Typically a frontend runs in $O(n)$ or $O(n\log n)$, while the back end is NP-complete.
## The frontend
![[w1n1frontend.png]]
The frontend consists of a number of stages:
1. The **scanner** converts a source into a stream of characters, and filters out comments and (usually) whitespace.
2. The **tokeniser** identifies words in the character stream and produces tokens, e.g. `x=y+2` becomes `IDENTIFIER(x) EQUAL IDENTIFIER(y) PLUS CST(2)`, and recognises some syntax errors.
3. The **parser** recognises context-free syntax and produces a parse tree, and detects more syntax errors. Parsers are often made using automatic parser generators.
4. The **semantic analyser** gives context-sensitive analysis, such as typechecking and ensures variables and functions are declared before they are used.
5. The **IR generator** produces the IR that is used by the rest of the compiler.
## The backend
![[w1n1backend.png]]
1. **Instruction selection** selects appropriate instructions for the program to use taking advantage of target architecture features like addressing modes or vectorisation.
2. **Register allocation** ensures all values are in a register when they are used, and selects what values are pushed to the stack when all registers are in use. Optimal allocation is an NP-complete problem, with compilers approximating solutions.
3. **Instruction scheduling** reorders instructions to optimiser [[W3N3 - RISC-V pipelined instruction execution|pipeline utilisation]] which improves performance.
# Optimising compilers
A compiler may have more stages, to optimise code before it is emitted. The optimiser will do things like:
- Propagate constant values, e.g. `x = 2; y = x + 1;` here, the second `x` can be substituted for 2, giving `y = 2 + 1;`, which can then further be simplified to `y = 3;`.
- Move infrequently executed computations out of the hot path
- Specialise computation based on context
- Remove redundant, useless, or unreachable code
- Encode specific idioms in efficient forms
# Compiler adjacent programs
There are a number of program types that are often used with compilers:
- **Preprocessors** perform textual substitution on source code before it is compiled, such as for `#define` or `#include` macros in C/C++
- **Assemblers** translate assemble language into binary
- **Linkers** link various compiled files and libraries together into one full executable program
