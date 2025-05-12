TimSort is the default sorting algorithm in python, which combines the advantages of [[W3N2 - Asymptotic analysis of sorting algorithms#InsertSort|InsertSort]] and [[W3N2 - Asymptotic analysis of sorting algorithms#MergeSort|MergeSort]].
It operates by first performing a step looking for "runs" of strictly decreasing or increasing items.
These can then be reversed/left unsorted.
The sorted runs can then be merged using MergeSort for long sub-arrays and InsertSort for shorter ones.