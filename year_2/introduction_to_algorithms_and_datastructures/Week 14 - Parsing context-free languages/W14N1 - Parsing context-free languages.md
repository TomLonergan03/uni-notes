The **parsing problem** is that of how we get from a [[W13N4 - Context-free languages and grammars|CFG]] sentence to a [[W13N4 - Context-free languages and grammars#Syntax trees|syntax tree]].
E.g. using a CFG with terminals `+,0...9,(,)` and non-terminals `Exp,Var` and rules:
`Exp -> Num | ( Exp + Exp )`
`Num -> 0 | ... | 9`
We have the string `( 3 + ( 4 + 5 ) )`. This is produced by the tree ![[w14n1ParsingTree.png]]
However, how do we do this algorithmicly?

# The Cocke-Younger-Kasami Algorithm
The **Cocke-Younger-Kasami (CYK) Algorithm** is an algorithm that can produce a syntax tree for any sentence in any CFG. This algorithm is another example of [[W11N2 - Dynamic programming|dynamic programming]].
- The algorithm works on a special class of grammars, those in    [[W14N2 - Chomsky normal form|Chomsky normal form (CNF)]]
	- This is still general as any CFG can be [[W14N2 - Chomsky normal form#Converting a grammar to CNF|transformed into an equivalent one]] in CNF
- CYK parses an input of length $n$ in $\Theta(n^3)$. This is fine for short sentences, but quickly becomes impractical for long computer programs. There we can instead use [[W14N3 - LL(1) Parsing|LL(1) parsing]], which is faster at the cost of being less general

# Parsing using the CYK algorithm
If we want to parse the string `my very heavy orange book`, we can insert position markers: `(0) my (1) very (2) heavy (3) orange (4) book (5)`. We can then talk about sub-strings, e.g $(2,4)$ indicates `heavy orange`.

The primary question we want to answer is: Can the entire string $(0,5)$ be derived from the start symbol `NP`? If so, how?

For the dynamic programming we generalise our objective slightly: Which sub-strings can be derived from which non-terminals?

The solutions to these sub-problems are stored in a 2D array, where entry `[i, j]` records possible sources of the sub-string $(i,j)$.

The table looks like this:

| `j` <br> `i` | 1<br>`my` | 2<br>`very` | 3<br>`heavy` | 4<br>`orange` | 5<br>`book` |
| -------- | ------- | --------- | ---------- | ----------- | --------- |
| 0 `my`     |         |           |            |             |           |
| 1 `very`   |         |           |            |             |           |
| 2 `heavy`  |         |           |            |             |           |
| 3 `orange` |         |           |            |             |           |
| 4 `book`       |         |           |            |             |           |

First, we can start with $i=0,j=1$, and `my` can be made in 1 way, from `Det`:

| `j` <br> `i` | 1<br>`my` | 2<br>`very` | 3<br>`heavy` | 4<br>`orange` | 5<br>`book` |
| ------------ | --------- | ----------- | ------------ | ------------- | ----------- |
| 0 `my`       | `Det`     |             |              |               |             |
| 1 `very`     |           |             |              |               |             |
| 2 `heavy`    |           |             |              |               |             |
| 3 `orange`   |           |             |              |               |             |
| 4 `book`     |           |             |              |               |             |

Next, $(1,2)$ can only be made by `Adv`:

| `j` <br> `i` | 1<br>`my` | 2<br>`very` | 3<br>`heavy` | 4<br>`orange` | 5<br>`book` |
| ------------ | --------- | ----------- | ------------ | ------------- | ----------- |
| 0 `my`       | `Det`     |             |              |               |             |
| 1 `very`     |           | `Adv`       |              |               |             |
| 2 `heavy`    |           |             |              |               |             |
| 3 `orange`   |           |             |              |               |             |
| 4 `book`     |           |             |              |               |             |

Next, we can produce both $(1,3)$ and $(2,3)$:

| `j` <br> `i` | 1<br>`my` | 2<br>`very` | 3<br>`heavy` | 4<br>`orange` | 5<br>`book` |
| ------------ | --------- | ----------- | ------------ | ------------- | ----------- |
| 0 `my`       | `Det`     |             |              |               |             |
| 1 `very`     |           | `Adv`       | `AP`         |               |             |
| 2 `heavy`    |           |             | `A,AP`       |               |             |
| 3 `orange`   |           |             |              |               |             |
| 4 `book`     |           |             |              |               |             |

Next, $i=0...3,j=4$ are all possible values:

| `j` <br> `i` | 1<br>`my` | 2<br>`very` | 3<br>`heavy` | 4<br>`orange` | 5<br>`book` |
| ------------ | --------- | ----------- | ------------ | ------------- | ----------- |
| 0 `my`       | `Det`     |             |              | `NP`          |             |
| 1 `very`     |           | `Adv`       | `AP`         | `Nom`         |             |
| 2 `heavy`    |           |             | `A, AP`      | `Nom`         |             |
| 3 `orange`   |           |             |              | `Nom, A, AP`  |             |
| 4 `book`     |           |             |              |               |             |

Lastly, we can fill out $i=0...4, j=5$:

| `j` <br> `i` | 1<br>`my` | 2<br>`very` | 3<br>`heavy` | 4<br>`orange` | 5<br>`book` |
| ------------ | --------- | ----------- | ------------ | ------------- | ----------- |
| 0 `my`       | `Det`     |             |              | `NP`          | `NP`        |
| 1 `very`     |           | `Adv`       | `AP`         | `Nom`         | `Nom`       |
| 2 `heavy`    |           |             | `A, AP`      | `Nom`         | `Nom`       |
| 3 `orange`   |           |             |              | `Nom, A, AP`  | `Nom`       |
| 4 `book`     |           |             |              |               | `Nom`       |

And as $(0,5)$ contains `NP`, which is $S$ for this CFG, this is a valid string in this CFG.

# Implementation
```
CYK(string, Grammar):
	n = len(string)
	table = [[] * n] * (n - 1)
	for j = 1 to n:
		for (X -> t) in Grammar: # all expansions to terminals
			table[j-1,j] = X
		for i = j - 2 to 0:
			for k = i + 1 to j - 1:
				for (X -> YZ) in Grammar: # all expansions to non-terminals
					if table[i, k] == Y && table[k, j] == Z:
						table[i, j] = X
	return table
```

This is, however, just a recogniser: it will determine if the string belongs to the given language, but not how it is constructed.

To change to a parser, it requires storing which existing constituents were combined to make each new constituent.
i.e. store `i,j,k` for each cell that is filled

This will give us a syntax tree based off of $G'$, however it is relatively simple to convert to a tree based off of $G$.

# Asymptotics
We have 3 nested for loops, each of which run $\leq n$ times, resulting in a runtime for a fixed grammar $G$ of $O(n^3)$, however if the grammar varies then iterating over the rules would also take $O(m)$ meaning an overall runtime of $O(mn^3)$.

If we allowed ternary rules, e.g. A -> BCD, we then have to consider all possible 3 way splits $(i,k),(k,l),(l,j)$ where $i<k<l<j$. This would result in a runtime of $\Theta(n^4)$.
This is the main reason why we use Chomsky normal form, as it minimises the runtime.

CYK is widely used in natural language processing, where sentences tend to be $< 100$ words. $Theta(n^3)$ is not typically good enough for computer languages.