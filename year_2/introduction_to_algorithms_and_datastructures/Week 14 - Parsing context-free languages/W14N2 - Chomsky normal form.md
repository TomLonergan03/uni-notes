In a [[W13N4 - Context-free languages and grammars|CFG]], the right side of each production is a (possibly empty) string of terminals and non-terminals. E.g.
`Exp -> ( Exp + Exp )`

A grammar in **Chomsky normal form** is one in which each right-hand side consists of either:
- Just 2 non-terminals
- Just 1 terminal

For an example CNF CFG to parse:
Terminals: `book, orange, heavy, my, very`
Non-terminals: `NP, Nom, AP, A, Det, Adv`
Start symbol: `NP`

`NP -> Det Nom`
`Nom -> book | orange | AP Nom`
`AP -> heavy | orange | Adv A`
`A -> heavy | orange`
`Det -> my`
`Adv -> very`

This will generate noun phrases like:
`my very heavy orange` and `my very heavy orange book`

CNF grammars often include duplication.
In the above case, `AP -> A | Adv A` would be simpler, but is not valid in CNF.

# Converting a grammar to CNF
A CFG $G=(\Sigma,N,S,P)$ is in CNF if all $P$ are of the form `A -> BC` or `A -> a` $(A,B,C\in N,a\in\Sigma)$

Theorem: Disregarding the empty string, every CFG $G$ is equivalent to a CNF $G'$ (or $L(G')=L(G)-\{\epsilon\}$), and there exists an algorithm which, given $G$, finds a suitable $G'$.

The basic idea is that, given a rule such as `A -> BCD`, this can be replaced by `A -> BY` and `Y -> CD`, where `Y` is a new non-terminal.

Consider the grammar
$S\rightarrow TT|[S]$
$T\rightarrow\epsilon|(T)$

### Step 1: Split rules with $\geq3$ symbols on RHS.
$S\rightarrow TT|[W$
$W\rightarrow S]$
$T\rightarrow \epsilon|(V$
$V\rightarrow T)$

### Step 2: identify set $E$ of all non-terminals $X$ such that $\epsilon$ can be derived from $X$ (nullable non-terminals)
$E=\{S,T\}$

### Step 3: delete all $\epsilon$-productions.
To compensate, for each rule $X\rightarrow Y\alpha$ or $X\rightarrow \alpha Y$, where $Y\in E$ and $\alpha\neq\epsilon$, add a new rule $X\rightarrow\alpha$.

In this case, $E=\{S,T\}$ so we get:
$S\rightarrow TT| T |[W$
$W\rightarrow S] | ]$
$T\rightarrow(V$
$V\rightarrow T)|)$

### Step 4: Remove unit productions $X\rightarrow Y$.
To compensate, for each rule $Y\rightarrow\alpha$ replace with $X\rightarrow\alpha$.

In this case, only $S\rightarrow T$ is the only case, so:
$S\rightarrow TT| (V |[W$
$W\rightarrow S] | ]$
$T\rightarrow(V$
$V\rightarrow T)|)$

Now, all RHSs consist of 1 terminal or 2 symbols.

### Step 5: Replace all terminals $a$ with non-terminal $Z_a$ and a production $Z_a\rightarrow a$.
In this case, we need 4 new rules:
$S\rightarrow TT| Z_(V |Z_[W$
$W\rightarrow SZ_] | ]$
$T\rightarrow Z_(V$
$V\rightarrow TZ_)|)$
$Z_(\rightarrow($
$Z_)\rightarrow)$
$Z_[\rightarrow[$
$Z_]\rightarrow]$

The grammar is now in CNF so we are done.

# Asymptotics
Given a grammar $G$ with $M$ rules, we can get a $G'$ with $O(m^2)$ rules.
Finding the smallest CNF grammar equivalent to $G$ is an [[W16N1 - P and NP#NP-hard|NP-hard]] problem.