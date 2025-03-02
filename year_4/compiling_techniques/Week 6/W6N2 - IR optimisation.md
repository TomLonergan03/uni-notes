![[w6n2optimiser.png]]
The optimiser produces IR which has the same behaviour as its input, but that performs that behaviour in a faster or more efficient way. There are many passes that can be performed, so we will look at two here.
# Constant folding
If all arguments to an operation are constants, we can perform the operation at compile time and store the resulting value at its output.

Given an expression:
```python
print(42 + 8)
```
we have the IR:
```
%0 = "choco.ir.literal"() <{"value" = 42 : !i32}>
%1 = "choco.ir.literal"() <{"value" = 8 : !i32}>
%2 = "choco.ir.binary_expr"(%0, %1) <{"op" = "+"}>
"choco.ir.call_expr"(%2) <{"func_name" = "print"}>
```
As 42 and 8 are constant values, we can perform the addition and add the result as a new variable:
```
%0 = "choco.ir.literal"() <{"value" = 42 : !i32}>
%1 = "choco.ir.literal"() <{"value" = 8 : !i32}>
%2 = "choco.ir.binary_expr"(%0, %1) <{"op" = "+"}>
%3 = "choco.ir.literal"() <{"value" = 50 : !i32}>
"choco.ir.call_expr"(%3) <{"func_name" = "print"}>
```
Notice that we now use `%3` as the argument to `print`, and that `%2` is no longer used (we will deal with that in a minute).

For a larger example, we start with
```python
42 + 8 + 5
```
and the IR:
```
%0 = "choco.ir.literal"() <{"value" = 42 : !i32}>
%1 = "choco.ir.literal"() <{"value" = 8 : !i32}>
%2 = "choco.ir.binary_expr"(%0, %1) <{"op" = "+"}>
%3 = "choco.ir.literal"() <{"value" = 5 : !i32}>
%4 = "choco.ir.binary_expr"(%2, %3) <{"op" = "+"}>
```
which is the graph 
![[w6n2beforeConstantFolding.png]]

First, we can fold `%2`, and get
```
%0 = "choco.ir.literal"() <{"value" = 42 : !i32}>
%1 = "choco.ir.literal"() <{"value" = 8 : !i32}>
%2 = "choco.ir.literal"() <{“value” = 50 : !i32}>
%3 = "choco.ir.literal"() <{"value" = 5 : !i32}>
%4 = "choco.ir.binary_expr"(%2, %3) <{"op" = "+"}>
```
Now, both arguments for `%4` are constants, so we can fold it:
```
%0 = "choco.ir.literal"() <{"value" = 42 : !i32}>
%1 = "choco.ir.literal"() <{"value" = 8 : !i32}>
%2 = "choco.ir.literal"() <{“value” = 50 : !i32}>
%3 = "choco.ir.literal"() <{"value" = 5 : !i32}>
%4 = "choco.ir.literal"() <{“value” = 55 : !i32}>
```
This is the same as the graph:
![[w6n2afterConstantFolding.png]]
# Dead code elimination
We are now left with many operations that are not used anywhere. Traversing the tree to remove them will prevent a lot of wasted effort at runtime.
# Pattern rewriting
Pattern rewriting uses rules to replace the current node with a new one if it matches a specific pattern.

With xDSL, we can do constant folding:
```python
@dataclass
class BinaryExprRewriter(RewritePattern):
	@op_type_rewrite_pattern
	def match_and_rewrite(self, expr: BinaryExpr, rewriter: PatternRewriter) -> None:
		if expr.op.data == '+':
			if isinstance(expr.lhs.op, Literal) and
			   isinstance(expr.rhs.op, Literal):
				lhs_value = expr.lhs.op.value.parameters[0].data
				rhs_value = expr.rhs.op.value.parameters[0].data
				result_value = lhs_value + rhs_value
				new_constant = Literal.get(result_value)
				rewriter.replace_op(expr, [new_constant])
		return
```
and to apply it to an entire module:
```python
def choco_flat_constant_folding(ctx: MLContext, module: ModuleOp) -> ModuleOp:

	walker = PatternRewriteWalker(GreedyRewritePatternApplier([
		BinaryExprRewriter(),
	]))
	
	walker.rewrite_module(module)
	return module
```

And dead code removal:
```python
@dataclass
class LiteralRewriter(RewritePattern):
	@op_type_rewrite_pattern
	def match_and_rewrite(self, literal: Literal, rewriter: PatternRewriter) -> None:
		if len(literal.results[0].uses) == 0:
			rewriter.replace_op(literal, [], [None])
		return
		
@dataclass
class BinaryExprRewriter(RewritePattern):
	@op_type_rewrite_pattern
	def match_and_rewrite(self, expr: BinaryExpr, rewriter: PatternRewriter) -> None:
		if len(expr.results[0].uses) == 0:
			rewriter.replace_op(expr, [], [None])
		return
```
and apply it to the module:
```python
def choco_dead_code_elimination(ctx: MLContext, module: ModuleOp)
-> ModuleOp:
	walker = PatternRewriteWalker(GreedyRewritePatternApplier([
		LiteralRewriter(),
		BinaryExprRewriter(),
	]), walk_reverse=True)
	walker.rewrite_module(module)
	return module
```