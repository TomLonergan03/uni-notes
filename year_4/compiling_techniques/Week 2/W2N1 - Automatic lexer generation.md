# Finite state automata
We can automatically generate a [[W1N2 - Lexical analysis#Lexers|lexer]] from a collection of regular expressions using **finite state automata (FSA)**. 

An FSA is defined by
- $S$: a finite set of states
- $\Sigma$: an alphabet (the character set the recogniser will use)
- $\sigma(s,c)$: a transition function which takes a state and a character and returns a new state
- $s0$: the initial state
- $SF$: a set of final states (a stream of characters is accepted iff the FSA ends up in any final state)

A regular expression corresponds to a recogniser/FSA, e.g.
`register ::= r(0|1|...|9)(0|1|...|9)*` is equivalent to
![[w2n1recogniser.png]]

The transition function has two inputs, the current state and an input.

| $\sigma$ | 'r'   | 0\|1\|...\|9 | all other inputs |
| -------- | ----- | ------------ | ---------------- |
| s0       | s1    | error        | error            |
| s1       | error | s2           | error            |
| s2       | error | s2           | error            |
A simple recogniser can be implemented as follows:
```python
c = next_character()

state = “s0”
while c != EOF:
	state = δ(s, c)
	c = next_character()
if (state final):
	return success
else:
	return error
```
# Non-determinism
If we have an RE such as `(a|b)*abb`, we get
![[w2n1nfa.png]]
which includes a $\epsilon$ transition, which may be followed without consuming an input character, and has two transitions out of $s1$ when an input 'a' is received. This makes it a **nondeterministic finite automata**, which would require backtracking (undesirable due to performance issues) during evaluation as we don't know which transition is required.
# Generating FSAs from REs
When we generate a lexer, we first generate the finite state automata.
- `x` becomes
  ![[w2n1renfax.png]]
- `[M]` becomes
  ![[w2n1renfam.png]]
- `M | N` becomes
  ![[w2n1renfaMN.png]]
- `M N` becomes
  ![[w2n1renfaMN.png]]
- `M+` becomes
  ![[w2n1renfaM+.png]]
# Converting NFAs to DFAs
If the generated FSA is nondeterministic we can then convert it to a deterministic finite automata. This is done by building a DFA which has one state for each set of states the NFA could end up in. A set of states in the DFA is final if it contains a final state in the NFA. Since the set of states in the NFA is finite ($n$), the number of possible sets of state is also finite (a maximum of $2^n$).

We have two key functions:
- $reachable(s_i,\alpha)$ returns the set of states reachable from $s_i$ when consuming character $\alpha$
- $closure(s_i)$ returns the set of states reachable from $s_i$ by $\epsilon$ (i.e. without consuming a character)
## Algorithm
The **subset construction algorithm** will convert an NFA to a DFA:
```
q0 = closure(s0); Q={q0}; worklist = [q0]; transitions = {};
while worklist not empty:
	remove q from worklist
	for each a in alphabet:
		subset = closure(reachable(q,a))
		transition[(q,a)] = subset
		if subset not in Q:
			add subset to Q and to worklist
```
This is the equivalent of:
1. Start from the start state $s_0$ of the NFA, compute its $\epsilon$-closure
2. Build a subset from all states reachable from $q_0$ for a character $\alpha$
3. Add this subset to the transition table
4. If the subset hasn't been seen before, add it to the work list
6. Repeat until no new subsets are created

E.g. for `a(b|c)*` we produce this NFA:
![[w2n1nfaToDfa.png]]

and can then create a table of states reachable from each NFA state and character input:

| DFA states | NFA states                      | a     | b     | c     | other |
| ---------- | ------------------------------- | ----- | ----- | ----- | ----- |
| $q_0$      | $s_0$                           | $q_1$ | none  | none  | none  |
| $q_1$      | $s_1,s_2,s_3,$<br>$s_4,s_6,s_9$ | none  | $q_2$ | $q_3$ | none  |
| $q_2$      | $s_5,s_8,s_9,$<br>$s_3,s_4,s_6$ | none  | $q_2$ | $q_3$ | none  |
| $q_3$      | $s_7,s_8,s_9,$<br>$s_3,s_4,s_6$ | none  | $q_2$ | $q_3$ | none  |
From which we can produce this DFA:
![[w2n1NfaToDfaResult.png]]
# Difficulties
Language design can complicate lexing:
- A language doesn't have reserved keywords results in much more complex parsing
- String constants may contain special characters (newlines, tabs, comment characters, quotation marks) which require special handling
# Overall
Combining this, we can automatically generate a lexer from a set of regular expressions, which are automatically turned into NFAs, then DFAs, then code is produced for the DFA.