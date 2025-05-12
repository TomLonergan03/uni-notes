We will use runtime cost, specifically the number of comparisons performed.

An algorithm will likely have differing performance depending on its inputs.

![[w3n2DifferingPerformance.png]]

Typically the best, worst, and sometimes average case times are discussed.

# InsertSort
```
0  InsertSort(A):
1	 for i = 1 to n−1 # write n for size of A  
2      x = A[i]  
3      j = i−1  
4      while j ≥ 0 and A[j] > x  
5        A[j+1] = A[j]  
6        j = j−1  
7      A[j+1] = x
```
The only comparison between elements occurs on line 4, at `A[j] > x` 
This will occur at most i times as we run from `j = i - 1` to `j = -1` at most.
`i` itself runs from `1` to `n-1`
Therefore the total number of `>` ops is at most $\sum^{n-1}_{i=1}i=O(n^2)$

## Worst case
Therefore InsertSort is $O(n^2)$, and will always perform $n^2$ or fewer comparisons on a list of length $n$. Is this attained?

Yes, given a list $[n,n-1,...,2,1]$ InsertSort's inner loop will always run until $j = -1$ so will perform exactly $\frac{n(n-1)}{2}$ comparisons. Thus we can say the worst case is $\Omega(n^2)$ and then $\Theta(n^2)$.

## Best case
The smallest number of comparisons InsertSort can perform is when it is provided an already sorted list. In this case the loop on line 4 will run once for each element of the list, resulting in $n-1$ comparisons, and therefore the best case is $\Theta(n)$.

We can therefore say that InsertSort is $O(n^2)$ and $\varOmega(n)$. 

# MergeSort
![[w3n2mergeSort.png]]
First we consider the **merge** operation. This will combine 2 sorted arrays $B$ and $C$ to produce $D$ ($|B|+|C|=m$).
A **merge** operation will perform at most $m-1$ operations, as each comparison results in a new element in $D$, and the last element is inserted without a comparison.
It will perform at least $min(|B|,|C|)$ if the shorter of the 2 is entirely inserted before the longer of the 2.
Therefore the number of comparisons done by **merge** is $\Theta(m)$. 

## Worst case
When analysing MergeSort we know that all comparisons will happen in the second half, once the elements have been split into individual 
lists. 
We can say then that there will be $n$ elements getting merged on each layer, to produce another list of length $n$, and that there will be $\text{lg }n$  total layers. Therefore the total number of comparisons = $m-1$ comparisons by **merge** x $\text{lg } n$ times. This is $O(n \text{ lg } n)$

## Best case
As **merge** is $\varTheta(m)$ regardless of the contents of $B$ and $C$ MergeSort will still perform $n \text{ lg } n$ comparisons. This is $\varOmega(n\text{ lg }n)$.

## Conclusion
MergeSort will always use $n\text{ lg }n$ comparisons regardless of the list.
Therefore MergeSort is $\varTheta(n\text{ lg }n)$ in it's best, average, and worst cases. This means that MergeSort is asymptotically better than InsertSort in worst and average cases, but performs worse in the best case, where the list is presorted.

# Space complexity
Doing an in-place InsertSort requires $\varTheta(1)$ extra space.
Doing MergeSort requires $\varTheta(n)$ extra space if after we do `D = Merge(B,C)` we then can reuse the space previously occupied by `B,C`.