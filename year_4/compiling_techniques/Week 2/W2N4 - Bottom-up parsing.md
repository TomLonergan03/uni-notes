# Bottom-up parsing
**Bottom-up parsing** builds a derivation by working from an input sentence back to the start symbol, as opposed to [[W2N2 - Top-down parsing|top-down parsing]] which works from a start symbol to the input sequence.

E.g, for the CFG:
$$
\begin{aligned}
Goal ::=\ &a\ A\ B\ e\\
A::=\ &A\ b\ c\ |\ b\\
B::=\ &d
\end{aligned}
$$
we parse the input $abbcde$ as follows:
$$
\begin{aligned}
&\text{a\textbf{b}bcde}\\
&\text{a\textbf{Abc}de}\\
&\text{aA\textbf{d}e}\\
&\text{\textbf{aABe}}\\
&\text{Goal}
\end{aligned}
$$
# Leftmost vs rightmost derivation
Leftmost derivations rewrite the leftmost non-terminal each iteration, while rightmost rewrites the rightmost non-terminal.
A top-down LL parser is equivalent to a bottom-up LR parser.
# Shift-reduce parser
A **shift-reduce parser** uses a stack along with four actions:
- `shift`: push the next input symbol onto the stack
- `reduce`: pop the symbols $Y_n,...,Y_1$ from the stack that form the right-hand side of a production rule $X::=Y_n,...,Y_1$
- `accept`: stop parsing and report a success
- `error`: stop parsing and report an error

E.g. for the same CFG, we do:

| Input    | Operation | Stack  | Note                                                                                                                                          |
| -------- | --------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| $abbcde$ | `shift`   | $a$    |                                                                                                                                               |
| $bbcde$  | `shift`   | $ab$   |                                                                                                                                               |
| $bcde$   | `reduce`  | $aA$   |                                                                                                                                               |
| $bcde$   | `shift`   | $aAb$  | Here, we don't know whether to<br>shift or reduce. We can use a DFA and<br>lookahead on the input to determine<br>when to `shift` vs `reduce` |
| $cde$    | `shift`   | $aAbc$ |                                                                                                                                               |
| $de$     | `reduce`  | $aA$   |                                                                                                                                               |
| $de$     | `shift`   | $aAd$  |                                                                                                                                               |
| $e$      | `reduce`  | $aAB$  |                                                                                                                                               |
| $e$      | `shift`   | $aABe$ |                                                                                                                                               |
|          | `reduce`  | $Goal$ |                                                                                                                                               |
|          | `accept`  |        |                                                                                                                                               |
# Top-down vs bottom-up parsing
Top-down parsing:
- Easy to write by hand
- Easy to integrate with the rest of the compiler
- Recursion can lead to performance issues

Bottom-up parsing:
- Very efficient
- Supports a larger class of grammars
- Requires generation tools
- Rigid integration with the rest of the compiler due to