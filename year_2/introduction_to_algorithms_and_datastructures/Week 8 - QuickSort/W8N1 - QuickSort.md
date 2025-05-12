QuickSort works by choosing a *pivot* element, then *partitioning*, which moves all elements in the array that are less than the pivot to come before it in the array. There are then 2 sub-arrays, which can be sorted recursively.

```
Partition(Array,start,end):
	pivot = Array[end]
	i = start - 1
	for j = start to end - 1:
		if Array[j] <= pivot
			i = i + 1
			swap Array[i] and Array[j]
	swap Array[i+1] and Array[end]
	
```
(this partition always uses the final element of the array range to partition, however you could use the start, middle, a random element, or something else)

## Asymptotics
Partition:
$T_{partition}(n)=\Theta(n)$

QuickSort:
$T_{QuickSort}=\Theta(n^2)$
however the average-case performance of QuickSort is $\Theta(n\text{ lg}(n))$.

QuickSort using the last element as pivot performs badly on sorted and nearly-sorted arrays, however this can be limited by choosing a different pivot, though the worst-case will remain $\Theta(n^2)$.