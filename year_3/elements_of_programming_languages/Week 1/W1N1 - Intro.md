# What is a programming language?
For the purpose of this course, a programming language is a formal, executable language for computations.
Examples of things that aren't programming languages:
- English (not formal)
- First order logic (formal, but not executable in general)
- HTML4 (formal, executable, but not computational)
- ChatGPT (executable, computational, but not formal although it can produce programs)

# Hello world in different languages

Python:
```python
print("Hello world!")
```

Java:
```Java
public class HelloWorld {
	public static void main(String[] args) {
		System.out.println("Hello world!");
	}
}
```

SQL:
```SQL
SELECT DISTINCT 'Hello world!' AS new_value FROM AnyTableWithOneOrMoreRows WHERE 1 = 1;
```

# Why are there so many?
Languages can be created with many different goals/motivations:
- Scientific computation: FORTRAN, R
- Commercial needs: COBOL, C, C++, Java, C#, F#, Ruby, JavaScript, Rust, SQL
- Scripting: Perl, Python, Ruby
- Exploring research ideas: LISP, Simula, Smalltalk, Algol, Pascal, Scheme, Racket, ML, OCaml, Haskell, Prolog, Curry

These can change over time, e.g. Python is now widely used for scientific computation

# What do they have in common?
- All formal languages have a written form, this is called the *syntax*
- All executable languages can be implemented on a computer, e.g. by a compiler or interpreter
- All programming languages describe computations: they have computational meaning (*semantics*)
- Most languages provide *abstractions* for organising, decomposing, and combining parts of programs to solve larger problems

# Paradigms
- Imperative
	- Variables, assignment, if/while/for, procedures
- Object oriented
	- Classes, inheritance, interfaces, subtyping
- Typed
	- Can be static, dynamic, strong, un/uni-typed
- Functional
	- $\lambda$-calculus, pure, lazy
- Logic/declarative
	- Query languages, computation as deduction
These overlap, and a language can be in multiple at once, e.g. C++ is object oriented and statically typed

# Elements of programming languages
There is no "best" paradigm, each has strengths and weaknesses. Across all paradigms there tend to be common features/elements. This course aims to understand the basic components across a variety of languages, and how they combine with one another

# Applicability
New major general purpose languages come along every decade or so. This is infrequent enough that most programmers will not need to design a general purpose language, however it is frequent enough that it is likely that a programmer will have to learn several new languages across their careers. In addition, domain-specific languages are increasingly used, and the same principles apply to them.

# Interpreters, compilers, and virtual machines
If we have a source language $L_S$, a target language $L_T$, and an implementation language $L_I$:
- An *interpreter* for $L_S$ is an $L_I$ program which executes $L_S$ programs
	- When both $L_S$ and $L_I$ are low-level (e.g. $L_S$ = JVM, $L_I$ = x86) and interpreter for $L_S$ is called a *virtual machine*
- A *translator* from $L_S$ to $L_T$ is an $L_I$ program that translates programs in $L_S$ to equivalent programs in $L_T$
	- When $L_T$ is low-level, a translator to $L_T$ is called a *compiler*

In this course we will use interpreters to explore different language features.

# Semantics
- How we understand the meaning of a language or feature, or compare different languages/features
- There are 3 basic approaches:
	1. *Operational semantics* defines the meaning of a program in terms of "rules" that explain the step-by-step execution of the program
	2. *Denotational semantics* defines the meaning of a program by interpreting it in a mathematical structure
	3. *Axiomatic semantics* defines the meaning of a program via logical specifications and laws
- All approaches have their own strengths and weaknesses
- We will focus on operational semantics as it is the most accessible and flexible approach

# Abstraction
The 3 most important considerations for a programming language design are:
- Data abstraction
- Control abstraction
- Modular abstraction

