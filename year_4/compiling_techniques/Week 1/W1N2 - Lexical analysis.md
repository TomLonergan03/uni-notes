![[w1n1frontend.png]]

The lexer maps a character stream into tokens.
# Context-free languages
A **context-free language** is specified with a grammar, usually written in **Backus Naur Form (BNF)**. A grammar $G$ consists of:
- $S$ - the starting symbol
- $N$ - a set of non-terminal symbols
- $T$ - a set of terminal symbols
- $P$ - a set of productions ($P:N\rightarrow N\cup T$)

E.g. defining addition and subtraction over numbers and identifiers:
$S$ = `goal`
$T$ = {`number`, `id`, `+`, `-`}
$N$ = {`goal`, `expr`, `term`, `op`}
$P$ = {
`goal` $\rightarrow$ `expr`,
`expr` $\rightarrow$ `expr op term` | `term`,
`term` $\rightarrow$ `number` | `id`,
`op` $\rightarrow$ `+` | `-`,
}
This grammar can then produce expressions such as `x + 2 - y`:
![[w1n2parseTree.png]]
# Regular expressions
A **regular expression (regex)** is a simplified form of BNF where:
- `x*` is the Kleene closure, which captures zero or more occurrences of `x`
- `x+` is the positive closure, which captures one or more occurrences of `x`
- `[x]` is an option, which captures zero or one occurrences of `x`
- Various ranges tend to be defined, e.g. `0-9` to capture any single digit, or `a-z`/`A-Z` for any lower/upper case character

E.g. defining signed integers is the regex: `0|([-](1-9)(0-9)*)`

A **regular language** any language which can be defined with a single regex or multiple non-recursive regexes, and we can automatically build recognisers from regular expressions.

We can use the following functions to create a lexer in python:
- `c` is the next character
- `next()` consumes the next character
- `error()` exits with an error message
- `first(exp)` returns the set of initial characters that match `exp`, e.g. for the signed integer regex `first("0|([-](1-9)(0-9)*)")` returns `{0,-,1,2,3,4,5,6,7,8,9}` as any of those options can be the first match of the regex

`x` can be matched with
```python
if c == x:
	next()
else:
	error()
```
`(exp)` is matched with
```python
pr(exp)
```
`[exp]` is matched with
```python
if c in first(exp):
	pr(exp)
```
`exp*` is matched with
```python
while c in first(exp):
	pr(exp)
```
`exp+` is matched with
```python
pr(exp)
while c in first(exp):
	pr(exp)
```
`fact_1 ... fact_1` is matched with
```python
pr(fact_1)
...
pr(fact_n)
```
`term_1 | ... | term_n` is matched with
```python
if c in first(term_1):
	pr(term_1)
elif …
	…
elif c in first(term_n):
	pr(term_n)
else
	error()
```

## Left parsable
The above rules can't parse all regular grammars, instead only covering **left-parsable** grammars. A grammar is left-parsable if:
- `term_1 | … | term_n`: terms don't share any initial symbols.
- `fact_1 … fact_n`: if `fact_i` contains the empty symbol then `fact_i` and `fact_(i + 1)` do not share any common initial symbols.
- `[exp], exp*`: the initial symbols of `exp` cannot contain a symbol which belongs to the first set of an expression following `exp`

# Lexers
The main role of the **lexer** is to read a part of the input and return a **lexeme/token**. Whitespace is usually ignored by the lexer, such as tabs, spaces, and newlines, and comments are also removed. 

A token consists of a token type and some other additional information.
```python
@datatype
class Token:
	kind: TokenKind
	value: Any = None
```
With some token classes being:
- Identifiers: matches foo, main, ...
- Numbers: matches 0, -12, 1000, ...
- String literals: "Hello world!", "a", ...
- Equals: ==
- Assign: =
- Plus: +
- Lpar: (

We can create a lexer for simple arithmetic expressions:
BNF syntax:
```
identifier ::= letter ( letter | digit )∗
digit ::= ”0” | . . . | ”9”
letter ::= ”a ” | . . . | ” z ” | ”A” | . . . | ”Z”
number ::= digit+
plus ::= ”+”
minus ::= ”−”
```
and write the lexer:
```python
from enum import Enum
from dataclasses import dataclass

class TokenClass(Enum):
	IDENTIFIER = 0
	NUMBER     = 1
	PLUS       = 2
	MINUS      = 3

@dataclass
class Token:
	type: TokenClass
	value: any = None

	def __repr__(self):
		return self.type.name + ((":" + str(self.value)) if self.value else "")

class Scanner:
	def __init__(self, stream):
		self.stream = stream
		self.buffer = None
		
	def peek(self):
		if not self.buffer:
			self.buffer = self.next()
		return self.buffer
	
	def next(self):
		if self.buffer:
			c = self.buffer
			self.buffer = None
			return c
		return self.stream.read(1)

class Tokenizer:
	def __init__(self, scanner):
		self.scanner = scanner
		self.buffer = None

	def peek(self):
		if not self.buffer:
		self.buffer = self.next()
		return self.buffer
	
	def next(self):
		if self.buffer:
			c = self.buffer
			self.buffer = None
			return c
		c = self.scanner.next()
		
		if c.isspace():
			return self.next()
			
		if c == "+":
			return Token(TokenClass.PLUS)
			
		if c == "-":
			return Token(TokenClass.MINUS)
			
		if c.isalpha():
			name = c
			c = self.scanner.peek()
			while c.isalpha() or c.isdigit():
				name += c
				self.scanner.next()
				c = self.scanner.peek()
			return Token(TokenClass.IDENTIFIER, name)
			
		if c.isdigit():
			digits = c
			c = self.scanner.peek()
			while c.isdigit():
				digits += c
				self.scanner.next()
				c = self.scanner.peek()
			value = int(digits)
			return Token(TokenClass.NUMBER, value)
```