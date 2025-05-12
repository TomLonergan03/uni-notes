An <b>efficient</b> algorithm is one that uses minimal time/space/instructions (depending on the use-case) to solve a problem. The most efficient algorithm for a problem is often (but not always) not the most obvious one.

### Example 1
##### Problem
Given a large decimal integer n, compute $n \text{ mod } 9$

##### Method A
Do full division and note the remainder.
![[w1n2DivisionAlgorithm.png]]

##### Method B
Add the digits of n to get a new number n'
Do the same with n' and repeat until we get a single digit d. If d = 9 then n mod 9 = 0, otherwise n mod 9 = d.

$\textit{For } n = 3748261728395607:$
$3+7+4+8+2+6+1+7+2+8+3+9+5+6+0+7 = 78$
$7 + 8 = 15$
$1 + 5 = 6$

**Why does this work?**
e.g. for a 4-digit number $abcd$:
$abcd = 1000a + 100b + 10c + d$
$=(999a+a)+(99b+b)+(9c+c)+d$
$=a+b+c+d\text{ mod } 9$

##### Comparing methods A and B
B has some advantages:
- It is faster, though not dramatically so
	- This is especially true for computers, for which addition is drastically faster than division
- It is more flexible, as the digits can be added in any order
	- It can therefore be parallelised by assigning adding different digits to different agents and summing their result once all complete their part

##### Moral
There may be non-obvious alternatives for an obvious algorithm that are faster and those alternatives will often need to be mathematically justified.

### Example 2
##### Problem
Given large whole numbers $a,n,m$ compute $a^n\text{ mod } m$. This is a fundamental component of modern cryptography.

##### Method A
Compute $a^n$ then $\text{mod } n$ 
This works for small numbers, but as soon as $n$ becomes large it becomes infeasible.
If e.g. $a = 3,$  $n = 123456789012345678901234$,  then $a^n$ won’t fit in memory.
Additionally, working with big numbers is time consuming.

##### Method B
Start from $a$.
Do $(n-1)$ multiplications by $a$, but reduce $\text{mod } m$ each time. This works because:
$(x\cdot y) \text{ mod } m = ((x \text{ mod } m) \cdot  (y \text{ mod } m)) \text{ mod } m$
e.g for $2^{10} \text{ mod } 17$:
$2\cdot 2=4,4\cdot 2=8,8\cdot 2=16,16\cdot 2=32\equiv15,15\cdot 2=30\equiv13$$13\cdot 2=26\equiv9,9\cdot 2=18\equiv1,1\cdot 2=2,2\cdot 2=4$
Now numbers will never be bigger than $am$.
This is still impractical if $n = 123456789012345678901234$

##### Method C
Notice that it is easy to compute $e=a^n\text{ mod } m$ if we've already computed $d=a^{floor(n/2)} \text{ mod } m$:
- If n is even, take $e = (d\cdot d) \text{ mod } m$
- If n is odd, take $e = (a\cdot d\cdot d) \text{ mod } m$
Therefore we can produce a **recursive algorithm**:
```
Expmod (a, n, m): # computes a^n mod m
	if n = 0
		return 1
	else
		d = Expmod (a, floor(n/2), m)
		if n is even
			return (d * d) mod m
		else
			return (d * d * a) mod m
```
##### Comparing the approach
![[w1n2ModAlgorithm.png]]
The time taken is significantly faster for method C than B or especially A. Not that for $n=10$ method B was faster than method C, however as $n$ increases B slowed down far faster than C did (this is because C is faster asymptotically, which we will come to later).

### Sorting arrays
##### InsertSort
We can sort an array by extracting a number and moving each of its right side neighbours along until the number is smaller than its neighbour.
```
InsertSort(A)
	for i = 1 to |A| - 1
		x = A[i]
		j = i - 1
		while j >= 0 and A[j] > x
			A[j + 1] = A[j]
			j = j - 1
		A[j + 1] = x
```

##### MergeSort
We can instead sort the array by splitting it in two halves, sorting the halves, then merging the results
```
Merge(B,C):
	D = [] \cdot  (|B| + |C|)
	i = j = 0
	for k = 0 to |D| - 1
		if B[i] < C[j]
			D[k] = B[i]
			i = i + 1
		else
			D[k] = C[j]
			j = j + 1
	return D
```

##### Relative performance
![[w1n2SortAlgos.png]]
Merge sort appears fundamentally better (with regard to runtime)
This relative performance can be precisely defined using [[W2N1 - Asymptotic analysis - o and ω]]