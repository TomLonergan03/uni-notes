The **type checking** process verifies and enforces the type system. The **type system** is defined by a set of formal rules that describe when a program is well-typed. To perform type checking, we process the syntactically well-formed program and apply the typing rules to check if every definition, statement, and expression conforms to the rule.
# Typing rules
A typing rule is written as:
$$
\frac{\vdots}{O\vdash e:T}\ \ \text{[Name]}
$$
Each rule has:
- A name
- Zero or more premises above the line
- A **typing judgement** (conclusion) below the line

A typing judgement states $O\vdash e:T$ that in environment $O$, the expression $e$ is well typed and has type $T$. The environment records the types of all variables and functions that are in scope when typechecking $e$.

For more detail on type derivations, see [[W2N3 - Types|here]].