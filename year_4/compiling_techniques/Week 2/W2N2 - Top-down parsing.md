The **parser** checks the stream of tokens produced by the lexer for grammatical correctness, determining if the input is syntactically well formed. It then builds a parse tree/**abstract syntax tree** for further semantic analysis.

Syntax is specified using [[W1N2 - Lexical analysis#Context-free languages|context free grammars]].
# Converting regular expressions to context free grammars
We can use some parts of regular expressions to make CFGs easier to define:
- `A*` can be represented by replacing `A*` with $Arep$ in all production rules, and adding a new production rule $Arep=A\ Arep\ |\ \epsilon$
- `A+` can be represented by replacing `A+` with $Apos$ in all production rules and adding a new production rule $Apos=A\ Apos\ |\ A$
- `[A]` can be represented by replacing `[A]` with $Aopt$, and adding a rule $Aopt=A\ |\ \epsilon$

E.g. if we define a grammar for a function call that uses a regular expression:
$$
\begin{aligned}
funcall&::=ID\ '('\ [\ ID\ (\ ','\ ID\ )\ *\ ]\ ')'\\
&\text{We can eliminate the option first:}\\
funcall&::=ID\ '('\ arglist\ ')'\\
arglist&::=ID\ (\ ','\ ID\ )\ *\ |\ \epsilon\\
&\text{then we can elimate the closure:}\\
funcall&::=ID\ '('\ arglist\ ')'\\
arglist&::=ID\ argrep\ |\ \epsilon\\
argrep&::=','\ ID\ argrep\ |\ \epsilon
\end{aligned}
$$
# Recursive descent parsing
To derive a syntactic analyser for a CFG in EBNF style:
- convert all regexes to production rules
- implement a function for each non-terminal symbol $A$, which recognises sentences derived from $A$

Recursion in the grammar then corresponds to recursive calls of the created functions.
## Implementation
```python
class Parser:
	...
	
	def check(self, expected : TokenKind) -> bool:
		return self.lexer.peek().kind == expected

	def match(self, expected : TokenKind) -> Token:
		if self.check(expected):
			token = self.lexer.peek()
			self.lexer.consume()
			return token
		raise Exception(f“Error: token of kind ${expected) not found”)
```

To parse the previous function call definition:
```python
	def parse_funcall(self):
		self.match(ID)
		self.match(LPAREN)
		self.parse_arglist()
		self.match(RPAREN)

	def parse_arglist(self):
		if self.check(ID):
			self.match(ID)
			self.parse_argrep()

	def parse_argrep(self):
		if self.check(COMMA):
			self.match(COMMA)
			self.match(ID)
			self.parse_argrep()
```
## Recursion
CFG productions cannot contain themselves as the left character (as then it would need expanded again, and would recurse infinitely), so should be rewritten to produce a leftmost terminal, e.g:
$$
\begin{matrix}
A&\rightarrow A\alpha_1|A\alpha_2|...|A\alpha_n|\beta_1|\beta_2|...|\beta_m&\text{(notice that $A\alpha_i$ produces a terminal $\alpha_i$ at the end)}\\
&\text{can be rewritten into}\\
A&\rightarrow\beta_1A^\prime|\beta_2A^\prime|...|\beta_mA^\prime&\text{(now, we still must start with a $\beta_i$ character,}\\
A^\prime&\rightarrow\alpha_1A^\prime|\alpha_2A^\prime|...|\alpha_nA^\prime|\epsilon&\text{which is still followed by any number of alpha characters)}
\end{matrix}
$$
## Lookahead
We can have grammars that have multiple cases with the same initial production. E.g.
```
stmt    ::= assign | funcall
funcall ::= ID "(" arglist ")"
assign  ::= ID "=" exp
```
We cannot parse this without backtracking, or for better performance we can look ahead to the next token to determine what production is correct.

In general, an arbitrary amount of lookahead is required, but large classes of CFGs can be parsed with limited lookahead, with most programming language constructs falling in those subclasses.
### LL(1)
A grammar has the LL(1) property if it:
- uses left-to-right parsing
- leftmost derivation, i.e. always apply a production to the leftmost non-terminal symbol
- requires only 1 current symbol to make a decision
### LL(k)
Sometimes we need to use $k>1$ tokens to determine which production to use. Our initial example grammar is an LL(2) grammar, as it requires 2 tokens to determine the correct production:
```python
def parse_stmt():
	if check([IDENT, EQ])
		parse_assign()
	if check([IDENT, LPAREN]):
		parse_funcall()
	error()
```