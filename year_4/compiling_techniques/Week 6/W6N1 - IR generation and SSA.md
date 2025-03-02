![[w6n1irGenerator.png]]
After a program is checked for [[W2N2 - Top-down parsing|syntactic]] and [[W4N1 - Semantic analysis|semantic]] errors, the compiler then starts optimising it. The frontend uses abstract syntax trees, but they are inconvenient for performing control-flow and data-flow analysis, which form the basis for many compiler optimisations. There are two key datastructures for these analyses: **def-use chains**, which provide the set of all uses for a single variable, and **use-def chains**, which provide the set of definitions for a use of a variable.

Our **intermediate representation (IR)** will want to be a graph-based structure of definitions and uses.
![[w6n1astToIr.png]]

# ChocoPy IR in xDSL
```python
@irdl_op_definition
class BinOp(Operation):
	name = "BinOp"
	op: OpAttr[StringAttr]
	lhs: Annotated[Operand, Attribute]
	rhs: Annotated[Operand, Attribute]
	result: OpResult
```
`Regions` represent nesting, but unlike in the AST, we only use nesting in the program, e.g. the then and else blocks of an if statement.
An example addition:
```
%l4 = Literal() ["value" = 4 : !i32]
%l5 = Literal() ["value" = 5 : !i32]
%res = BinaryExpr(%l4, %l5) ["op" = "+"]
```
and an if statement
```
%t = Literal() ["value" = !bool<True>]
If(%t) {
	%l4 = Literal() ["value" = 4 : !i32]
	%l8 = Literal() ["value" = 8 : !i32]
} {
	%l15 = Literal() ["value" = 15 : !i32]
	%l16 = Literal() ["value" = 16 : !i32]
}
```

Operands and results of operations are written with a percent sign before a name or number: `%x` or `%23`. Each name represents a value of a certain type. Operations expect their operands to be of a certain type, similar to a function argument.
# Single static assignment
Mutating variables causes problems during IR optimisation, so we remove them by instead introducing a new variable every time a value is assigned. This is **single static assignment (SSA)**, defined as a program in which every variable is the target of exactly one assignment statement in the program text.

With mutation:
```python
x = 1
y = x + 1
x = 2
z = x + 1
```
and without:
```python
x1 = 1
y = x1 + 1
x2 = 2
z = x2 + 1
```

