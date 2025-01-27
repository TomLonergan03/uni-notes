A parser builds a syntax tree, which is either:
- a concrete syntax tree
- an abstract syntax tree
# Concrete syntax tree
A **concrete syntax tree** directly corresponds to the parser's grammar.

E.g. in EBNF:
```
Expr   ::= Term ( (‘+’ | ‘-’) Term)*
Term   ::= Factor ( (‘*’ | ‘/’) Factor)*
Factor ::= number | ‘(‘ Expr ‘)’
```
and with the EBNF syntax removed:
```
Expr ::= Term Terms
Terms ::= (‘+’ | ‘-’) Term Terms | ε
Term ::= Factor Factors
Factors ::= (‘*’ | ‘/’) Factor Factors | ε
Factor ::= number | ‘(‘ Expr ‘)’
```
This will parse `3 * (4 + 5)` to
![[w3n1concreteSyntaxTree.png]]
This contains unnecessary entries, such as the `term-factor-number` chains.
# Abstract syntax tree
An **abstract syntax tree** abstracts away some of the details.

E.g. we can create a simpler grammar for arithmetic expressions:
```
Expr  ::= BinOp | intLiteral
BinOp ::= Expr Op Expr
Op    ::= add | sub | mul | div
```
which can then parse `3 * (4 + 5)` as:
![[w3n1abstractSyntaxTree.png]]
## Implementation
The AST is implemented like any other tree:
```python
class Expr(ABC):
	pass

@dataclass
class BinOp(Expr):
	lhs: Expr
	op: str
	rhs: Expr

@dataclass
class IntLiteral(Expr):
	value: int
```
this then allows us to build trees like:
```python
BinOp(IntLiteral(3), "*", BinOp(IntLiteral(4), "+", IntLiteral(5)))
```
## xDSL
**xDSL** is a framework for building compilers which is based on the MLIR framework.
We can define ASTs using xDSL:
```python
@irdl_op_definition
class BinOp(IRDLOperation):
	name = "BinOp"
	op = prop_def(StringAttr)
	lhs = region_def()
	rhs = region_def()

@irdl_op_definition
class IntLiteral(IRDLOperation):
	name = "IntLiteral"
	value = prop_def(IntegerAttr[IntegerType])
```
Every xDSL operation (node) inherits from `IRDLOperation`, and has a name, and possibly some attributes and/or regions. `region_def()` and `prop_def()` are macros that generate the code for regions and properties. Properties represent compile-time metadata, such as literal values, names of variables, etc, and regions represent nested code, e.g. `if then else` has a region for the `if` condition, a region for the `then` block, and a region for the `else` block.