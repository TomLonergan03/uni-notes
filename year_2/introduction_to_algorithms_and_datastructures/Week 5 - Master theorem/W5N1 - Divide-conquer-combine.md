# Recursion
A common thread through many algorithms is recursion, where an algorithm involves a call to itself.
E.g. `Expmod(a,n,m)` will then call `Expmod(a,floor(n/2),m`

A common pattern in recursion is to divide a large problem into smaller sub-problems, then "conquering" (solving) these smaller problems before combining the results.

# How to calculate asymptotics for recursive algorithms or The Master Theorem
If our recurrence relation is of the form:
$$\begin{align}
	T(n) = &\{\Theta(1) &\text{ if } n\leq n_0\\
	\text{or }&\{aT(n/b)+\Theta(n^k) &\text{ if } n>n_0
\end{align}$$
The answer depends how $a$ compares to $b^k$, or how $e=log_b\text{ }a$ compares with $k$

| Condition | $T(n)$                   |
| --------- | ------------------------ |
| $e>k$     | $\Theta(n^e)$            |
| $e=k$     | $\Theta(n^klg\text{ }n)$ |
| $e<k$     | $\Theta(n^k)$            |

## Example
Given the MergeSort recurrence:
$$
\begin{align}
	T(n)=&\{\Theta(1) &\text{if }n=1\\
		&\{2T(n/2)+\Theta(n) &\text{otherwise}
\end{align}
$$
So $a=2,b=2,k=1$
We can then calculate $e=log_2 2=1$ so $e=k$
Therefore MergeSort is $\Theta(n\text{ lg }n)$
