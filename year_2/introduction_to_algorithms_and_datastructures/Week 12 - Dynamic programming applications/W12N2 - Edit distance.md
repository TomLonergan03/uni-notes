The **edit distance** between 2 strings $s$ and $t$ over an alphabet $\Sigma$ is the number of insertion, deletion, and substitution operations required to change $s$ into $t$.
This is often used for DNA or RNA strings, or for words from a natural language.
e.g.
`k i t t e n -`
`s i t t i n g`
3 operations: substitute `k` for `s`, `e` for `i`, and insert `g`.

An alignment of 2 sequences $s\in\Sigma,t\in\Sigma$ is any padding (inserting `-`) $s'$ of $s$, and $t'$ of $t$, such that
$$
\begin{align}
	|s'|&= |t'|\\
	(s'_i\neq -)&\vee (t'_i\neq -)
\end{align}
$$
Often, there will be multiple possible alignments, such as for the DNA sequences:
`A C C G G T A T C C T A G G A C`
`A C C T A T C T - - T A G G A C`

`A C C G G T A T C C T A G G A C`
`A C C - - T A T C T T A G G A C`

These are both valid alignments, however the first one has an edit distance of 5, while the second has an edit distance of 3.

The score of an alignment is the total number of insertions $(s'_i\in\Sigma,t'_i=-)$, deletions $(s'_i=-,t'_i\in\Sigma)$, and substitutions $(s'_i \neq t'_i,s'_i\in\Sigma,t'_i\in\Sigma)$.

# Implementation

When finding the minimum edit distance between 2 strings, we need to find the optimal alignment of those strings. We know that there are only 4 possible alignments of the final column: an insertion, a deletion, a substitution, or a match.
For each of these cases, we can then consider the remainder of the strings by repeating this final column alignment.
We get a recurrence for the edit distance for $s=s[1...m],t=t[1...m]$:
$$
d(s[1...m],t[1...n]) = \left\{\begin{matrix}
	m & \text{if }n=0\\
	n & \text{if }n=0\\
	d(s[1...m-1],t[1...n-1]) & \text{if } s_m=t_n\\
	1+min(d(s[1...m-1],t[1...n-1]),\\
	d(s[1...m-1],t[1...n]), & \text{if } s_m\neq t_n\\
	d(s[1...m],t[1...n-1]))\\
\end{matrix}
\right.
$$
If we do this recursively, we have an exponentially sized tree. There are, however, only $m\cdot n$ sub-problems, so [[W11N2 - Dynamic programming|dynamic programming]] can be used.

We will need a array, `d`, of size $(m+1)\cdot(n+1)$.
- An entry `d[i,j]` is used to store the value of $d(s[1...i],t[1...j])$, once it has been computed.
- We need to ensure that `d[i-1,j-1]`, `d[i-1,j]`, `d[i,j-1]` have already been computed before we try to compute `d[i,j]`.
We will also keep an array, `a`, storing whether the optimum for $s[1...i],t[1...j]$ ended in a match, substitution, insertion, or deletion (represented in this case by 0, 1, 2, 3 respectively).
`a` will help us reconstruct the actual optimal alignment.

```
EditDistance(s[1...m], t[1...n]):
	for i = 0 to m:
		d[i, 0] = i
		a[i, 0] = 3
	for j = 0 to n:
		d[0, j] = j
		a[0, j] = 2
	for i = 1 to m:
		for j = 1 to n:
			if s[i] = t[j]:
				d[i, j] = d[i - 1, j - 1]
				a[i, j] = 0
			else
				d[i, j] = 1 + min(d[i, j - 1], d[i - 1, j], d[i - 1, j - 1])
				if d[i, j] = d[i - 1, j - 1] + 1:
					a[i, j] = 1
				else if d[i, j] = d[i, j - 1] + 1:
					a[i, j] = 2
				else:
					a[i, j] = 3
```

To reconstruct the best alignment:
```
ReconstructAlignment(a, s[1...m], t[1...m]):
	i = m
	j = n
	s' = ""
	t' = ""
	while i != 0 || j != 0:
		if a[i, j] == 0 || a[i, j] == 1:
			s'.append(s[i])
			t'.append(t[j])
			i -= 1
			j -= 1
		else if a[i, j] = 2:
			s'.append("-")
			t'.append(t[j])
			j -= 1
		else if a[i, j] = 3:
			s'.append(s[i])
			t'.append("-")
			i -= 1
	while i != 0:
		s'.append(s[i])
		t'.append("-")
		i -= 1
	while j != 0:
		s'.append("-")
		t'.append(t[j])
		j -= 1
	print(s'.reverse())
	print(t'.reverse())
```
(this should be right, I haven't tested it)

# Asymptotics
`EditDistance` contains a loop that is $O(m)$, then one that is $O(n)$, then finally a nested loop that comes to $O(m\cdot n)$, resulting in an overall runtime of $O(m\cdot n)$. It requires space of $O(m\cdot n)$.

`ReconstructAlignment` contains a loop that runs from 0 to $m$, and 0 to $n$ so is $O(m+n)$.

