Often, a problem can be solved using a recursive algorithm, such as when [[W5N1 - Divide-conquer-combine|divide and conquer]] is used.
This can give an efficient result, e.g. [[W3N2 - Asymptotic analysis of sorting algorithms#MergeSort|MergeSort]] and [[W8N1 - QuickSort|QuickSort]], however is not always efficient.

For an example of an Inefficient application of recursion, if we calculate the Fibonacci numbers by recursing to the base cases.
```
RecursiveFibonacci(n):
	if n == 0:
		return 0
	if n == 1:
		return 1
	return RecursiveFibonacci(n - 1) + RecursiveFibonacci(n - 2)
```

This will be incredibly slow for large $n$, as every non base-case call to `RecursiveFibonacci` results in 2 further `RecursiveFibonacci` calls.
This results in $O(2^n)$ run times.

We can instead use dynamic programming to calculate Fibonacci numbers.
```
DynamicFibonacci(n):
	F = [0] * (n + 1)
	F[1] = 1
	for i in 2 to n:
		F[i] = F[i - 1] + F[i - 2]
	return F[n]
```

This case runs in $\Theta(n)$ time, and requires $\Theta(n)$ storage space for the array.


# Principles
A dynamic programming approach is suitable when:
1. Computing the optimal solution can be achieved by finding solutions to smaller problems of the same type and combining them.
2. The solution is expressible in terms of a recurrence, where the right side contains one or more recursive calls to solve a smaller version of the same problem.
3. Storing results of all possible sub-problems must be possible and polynomially bounded.
4. The sub-problems on the right side of the recurrence must be computed in advance of computing the left side. This must be controllable by an algorithm.