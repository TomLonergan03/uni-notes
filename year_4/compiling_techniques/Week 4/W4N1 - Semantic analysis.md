The [[W2N2 - Top-down parsing|parser]] ensures that the input program is syntactically well-formed, i.e. an AST can be built for it. The **semantic analyser** then checks that the semantics of the AST is correct, i.e. the program has a well defined meaning.
# Syntax vs semantic errors
Syntax errors include:
```python
def foo():
4 + 3 # incorrect indentation

def foo{}: # wrong braces used
	4 + 3

def foo():
	4 plus 3 # multiple expressions on the same line
```
while semantic errors include:
```python
def foo():
	x + 3 # x is undeclared

def foo():
	"4" + 3 # can't add a string and an int

def foo():
	foo(3) # foo doesn't expect an argument
```
# Semantic analysis
Semantic analysis detects semantic errors. We will look at 3 different analyses which each check for one kind of semantic error:
- **Assign target analysis**: checks the left-hand side of an assignment is a valid target
- **Name analysis**: check all names of variables and functions are declared before they are used
- **Type analysis**: check the program is well-typed given a set of typing rules

Each semantic analysis is implemented as a unique pass which traverses the AST.
## AST visitors
An AST visitor visits each node in the tree, and performs an operation on each node. A simple AST visitor (using an [[W3N1 - Abstract Syntax#xDSL|xDSL]] AST) looks like:
```python
class Visitor:
	def traverse(self, operation: Operation):
		for r in operation.regions:
			for op in r.ops:
				self.traverse(op)
		self.visit(operation)

	def visit(self, operation: Operation): # overloaded by subclasses
		pass
```
We can then make a visitor that just prints the name of each operation:
```python
class SimplePrinter(Visitor):
	def visit(self, operation: Operation):
		print(operation.name) # print operation name

SimplePrinter().traverse(BinaryExpr.get(...))
```
In reality, we probably want to perform different operations on different AST node types, and we can use Python's dynamic reflection for the general case:
```python
class Visitor:
	def traverse(self, op: Operation):
		# get class name of operation in snake_case
		op_class_name = camel_to_snake(type(op).__name__)
		for r in op.regions:
			...
		# check if subclass has implemented a method with name visit_op_class_name
		# return method if it exists; otherwise None is returned
		visit = get_method(self, f"visit_{op_class_name}")
		if visit:
		visit(op) # if the visit_op_class_name method exists call it
```
and we can also generalise traversal to allow for flexible traversal:
```python
class Visitor:
	def traverse(self, op: Operation):
		class_name = camel_to_snake(type(op).__name__)
		traverse = get_method(self, f"traverse_{class_name}")
		if traverse: # if a traverse_class_name method
		# exists call it
			traverse(operation)
		else: # otherwise do the generic traversal
			for r in op.regions:
				...
		visit = get_method(self, f"visit_{class_name}")
		if visit:
			visit(op)
```
## Assign target analysis
We want to check that the left-hand side of all assignments is either:
- A variable name e.g. `x = 4 + 5`
- An index into a list e.g. `x[0] = 4 + 5`

In xDSL, this looks like:
```python
def check_assign_target(_: MLContext, module: ModuleOp) -> ModuleOp:
	class AssignVisitor(Visitor):
		# visit every assign AST node
		def visit_assign(self, assign: Assign):
			# select the target operation
			target_op = assign.target.ops[0]
			# check if it is a variable name or an index expression
			if isinstance(target_op, ExprName):
				return
			if isinstance(target_op, IndexExpr): 
				return
			# if not: raise a Semantic Error
			raise SemanticError(
				f'Found {type(target_op).__name__} as the left-hand side of an assignment. '
				f'Expected to find variable name or index expression only.')
		AssignVisitor().traverse(module)
		return module
```
## Name analysis
To check if variables and functions are declared before they are used, we need to keep track of a context which contains which names are currently assigned in each scope of the program.
### Scopes
The **scope** of an identifier is the part of a program in which that identifier is valid.
- It is only legal to refer to an identifier within its scope.
- It is illegal to declare two identifiers with the same name and the same scope
- It is legal to declare a variable in a nested scope, which then shadows the identifier in the outside scope which can no longer be accessed.
- Variables not declared within a function have **global scope**.
### Name analysis in practice
We have to know all the function names that are declared in a given scope, regardless of where in that scope the function is defined. This requires making two passes, the first to build the contexts and the second to check names.
The context builder looks like:
```python
class BuildContextVisitor(Visitor):
	name_ctx: NameCtx # class to manage the name context
	
	# for every variable definition
	def visit_var_def(self, var_def: VarDef): # add variable name to the current name context
		self.name_ctx.add_var(var_def.typed_var.ops[0].var_name.data)

	# for every function definition
	def traverse_func_def(self, func_def: FuncDef):
		# prepare a visitor for the function body …
		body_visitor = BuildContextVisitor(NameCtx(parent_scope=self.name_ctx))
		# … add the function parameters to the nested name scope …
		for op in func_def.params.ops:
			body_visitor.name_ctx.add_var(op.var_name.data)
		# … visit the function body to construct the nested name scope.
		for op in func_def.func_body.ops:
			body_visitor.traverse(op)
		# finally, add function and nested scope to the current name context
		self.name_ctx.add_func(func_def.func_name.data, body_visitor.name_ctx)
```
While the name checker looks like:
```python
class NameAnalysisVisitor(Visitor):
	name_ctx: NameCtx

	# check each variable has been declared before its used
	def visit_expr_name(self, expr_name: ExprName):
		if expr_name.id.data in self.name_ctx:
			return
		else:
			raise SemanticError(
				f'[Name Analysis Error]: '
				f"Identifier `{expr_name.id.data}' found that was not previously defined.")

	# check that functions are declared before they are called
	def visit_call_expr(self, call_expr: CallExpr):
		if call_expr.func.data in self.name_ctx:
			return
		else:
			raise SemanticError(
				f'[Name Analysis Error]: '
				f"Identifier `{call_expr.func.data}' found that was not previously defined.")

	# use a function's context when checking its body
	def traverse_func_def(self, func_def: FuncDef):
		# select the nested name context from the current name context …
		nested_ctx = self.name_ctx.get_func_ctx(func_def.func_name.data)
		# … and use the nested name context when traversing the function body
		body_visitor = NameAnalysisVisitor(nested_ctx)

		for op in func_def.func_body.ops:
			body_visitor.traverse(op)

	# check that the iteration variable in a for loop was previously defined
	def visit_for(self, for_op: For):
		if for_op.iter_name.data not in self.name_ctx:
			raise SemanticError(
				f'[Name Analysis Error]: '
				f"Identifier `{for_op.iter_name.data}' found that was not previously defined.")

	# check variables in global declarations exist in the global scope
	def visit_global_decl(self, global_decl: GlobalDecl):
		if global_decl.decl_name.data in self.name_ctx.global_scope():
			return
		else:
			raise SemanticError(
				f'[Name Analysis Error]: '
				f"Identifier `{global_decl.decl_name.data}' not declared in global scope.")
```

And finally putting the parts together:
```python
def name_analysis(_: MLContext, module: ModuleOp) -> ModuleOp:
	# add print, len, and input functions to the global scope
	name_ctx = NameCtx()
	name_ctx.add_func("print", NameCtx())
	name_ctx.add_func("len", NameCtx())
	name_ctx.add_func("input", NameCtx())
	
	# first construct name context
	BuildContextVisitor(name_ctx).traverse(module)
	
	# then perform checking
	NameAnalysisVisitor(name_ctx).traverse(module)
	
	return module
```