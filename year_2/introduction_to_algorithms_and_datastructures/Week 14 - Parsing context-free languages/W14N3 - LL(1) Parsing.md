If we define a simple programming language as a [[W13N4 - Context-free languages and grammars|CFG]]:
`stmt -> if-stmt | while-stmt | begin-stmt | assg-stm`
`if-stmt → if bool-expr then stmt else stmt` 
`while-stmt → while bool-expr do stmt`
`begin-stmt → begin stmt-list end`
`stmt-list → stmt | stmt ; stmt-list`
`assg-stmt → var := arith-expr`
`bool-expr → arith-expr compare-op arith-expr`
`compare-op → < | > | <= | >= | == | ! =`
Start symbol: `stmt`

We want to read from left to right, processing each token only once as we are aiming for $O(n)$ runtime.

We want to construct a leftmost derivation (expanding the leftmost non-terminal at each step).

If the first token in the program is `begin` then we know that the first two steps must be:
`stmt -> begin-stmt` 
`-> begin stmt-list end`
We can now parse the program as `begin stmt-list end`, and as begin is a terminal we can parse the remaining input as `stmt-list end`. This is the **predicted form** for the remaining input.

If it is always possible to determine the next production from this information, then the grammar is $LL(1)$.

The next step is expanding `stmt-list`, but if the next token is `if` then we have 2 rules that may apply: `stmt-list → stmt` or `stmt-list -> stmt ; stmt-list`. This shows that our grammar is not $LL(1)$, as there is no way to know which of these rules apply without looking further down the input string for a `;`.
In this case, we can rewrite the rules for `stmt-list` to fix the problem:
`stmt-list -> stmt stmt-tail`
`stmt-tail -> ε | ; stmt stmt-tail`

Now, when we see `if` we can see that the next two rules are:
`stmt-list -> stmt-tail`
`stmt -> if-stmt`
`if-stmt -> if bool-expr then stmt else stmt`

Therefore, the whole derivation so far is:
`stmt -> begin-stmt` 
`-> begin stmt-list end`
`-> begin stmt stmt-tail end`
`-> begin if-stmt stmt-tail end`
`-> begin if bool-expr then stmt else stmt stmt-tail end`

This accounts for the first two tokens, `begin if`. The predicted form for the rest is `bool-expr then stmt else stmt stmt-tail end`.

# Parse tables
If we have a grammar for bracket sequences:
$S\rightarrow\epsilon|TS$
$T\rightarrow(S)$
This is LL(1) so we can always tell from the current token and current non-terminal which rule to apply.

This means we can draw a 2D parse table, which states what rule should be applied in any situation. We also have an "end-of-input" marker $.

|     | (                  | )                       | $                       |
| --- | ------------------ | ----------------------- | ----------------------- |
| $S$ | $S\rightarrow TS$  | $S\rightarrow \epsilon$ | $S\rightarrow \epsilon$ |
| $T$ | $T\rightarrow (S)$ |                         |                         |

An entry in column $a$ and row $X$ is what rule is applied when $a$ is inputted and $X$ is the current predicted token.
An empty cell is a situation that cannot arise for a legal input.

## Example using the table
Using the above table, we can parse the input string `(())`.
We can keep track of the remaining predicted form using a [[W4N1 - Classic datatypes#Stacks and queues|stack]].
When we have a non-terminal at the front of the stack we do a lookup operation, which consumes that symbol from the stack but doesn't affect the remaining input. This throws an error if it hits a blank cell in the table.
When we have a terminal at the front of the stack we do a match operation, which consumes the front of the stack and the first token in the remaining input. This throws an error if the token doesn't match the top element in the stack.

| Operation   | Remaining input | Stack state |
| ----------- | --------------- | ----------- |
|             | (())$           | S           |
| Lookup (, S | (())$           | TS          |
| Lookup (, T | (())$           | (S)S        |
| Match (     | ())$            | S)S         |
| Lookup (, S | ())$            | TS)S        |
| Lookup (, T | ())$            | (S)S)S      |
| Match (     | ))$             | S)S)S       |
| Lookup ), S | ))$             | )S)S        |
| Match )     | )$              | S)S         |
| Lookup ), S | )$              | )S          |
| Match )     | $               | S           |
| Lookup $, S | $               | Empty |

The first operation, lookup (,S checks column (, row S in the table and finds $S\rightarrow TS$. This is then applied to the leftmost symbol in the stack, and changes the stack from S to TS.

The third operation, match (, sees that there is a ( at the front of the input and on top of the stack and consumes it from both, as it is the leftmost terminal and can now be ignored.

As the final operation, lookup $, S, replaces the final S with $\epsilon$ and consumes both the last input token and empties the stack, the input is legal.

It is also simple to construct a [[W13N4 - Context-free languages and grammars#Syntax trees|syntax tree]] by adding the result of the the operation found by the lookup as children of the leftmost item of the stack.

## Parsing invalid strings
Lets say we try to parse the string `)` using our CFG.

| Operation   | Remaining input | Stack state |
| ----------- | --------------- | ----------- |
|             | )$               | S           |
| Lookup ), S | )$               | Empty       |

At this point, the stack is empty but there is still remaining input tokens. This is therefore an illegal string.

If we are to try to parse the string `(` :
| Operation   | Remaining input | Stack state |
| ----------- | --------------- | ----------- |
|             | ($              | S           |
| Lookup (, S | ($              | TS          |
| Lookup (, T | ($              | (S)S        |
| Match (     | $               | S)S         |
| Lookup $, S | $               | )S          |

Now, as there is a terminal at the front of the stack, the next operation should be a match, however $ and ) are not equal, so this is an illegal string.

# Implementation
Returns `true` if successful, `false` if unsuccessful
```
LL1_Parse(table, start_symbol, input_string):
	position = 0
	stack = stack()
	stack.push(start_symbol)
	while !stack.isEmpty():
		x = stack.pop()
		if x is non-terminal:               # lookup case
			case table[x, input[pos]]:
				blank: return false
				rule x -> y:
					stack.push(reverse(y))
		else:                               # match case
			if x == input[pos]:
				pos += 1
			else:
				return false
	if input[pos] == $:
		return true
	else:
		return false
```

# Asymptotics
As each symbol is pushed once and popped once, the runtime is $O(n)$.
The parse table will use space $O(m\cdot n)$ where $m$ is the number of possible input tokens and $n$ is the number of possible symbols.

# Other notes
- LL(1) is a top-down parser, which builds the syntax tree from the root to the leaves. [[W14N1 - Parsing context-free languages#The Cocke-Younger-Kasami Algorithm|CYK]], however, is a bottom-up parser.
- Not every context-free language has an LL(1) grammar, however if we are designing one (such as for a programming language) we can try to ensure that it does.
- LL(1) is nice for simple languages, however for larger scale languages other grammars are used, a common choice being LR(1) parsing.
- In the real world, we wouldn't implement a parser ourselves. We would instead define the CFG, then use a parser generator to automatically construct a parse table.

# In context
When we process a Java program, this is the sequence:
![[w14n3JavaProcessing.png]]

For spoken English, we have a similar sequence:
![[w14n3EnglishProcessing.png]]
Though English has constant ambiguity throughout, and the later stages feed back into the earlier ones, such as through and threw have the same phonemes, but different meaning which is only identifiable after segmentation and tagging have happened.