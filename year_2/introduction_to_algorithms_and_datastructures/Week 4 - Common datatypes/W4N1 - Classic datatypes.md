# Lists
A list can be implemented in a number of ways:

## Fixed length lists
Create an array with a fixed size.
In this case, getting and setting values, as well as length take $\varTheta(1)$ time.
Insert and delete take $\varTheta(n)$ time as every element after that must be shuffled forwards or backwards.

These have strengths:
- Fast get and set
- Fixed size means can go on the stack
and weaknesses:
- The length of the list is preset
- If list length is uncertain we have a risk of either allocating more memory than needed or overflowing the allocated memory

## Extensible array
We use a fixed length list, but once we reach the maximum size of the list, we simply copy the array into a new larger array.
This means that a normal append takes $\varTheta(1)$ time, however occasionally there will be an append that takes $\varTheta(n)$ time as the entire array gets copied.
Most programs are not so timing dependent that this occasional increase of time doesn't matter, however this does mean that append is actually $\varTheta(n)$ instead of $\varTheta(1)$.

### Calculating amortised cost of appends
If we have an array of capacity $a$ and every time we reach the limit we increase it by a factor of $r$, then performing $m$ appends will involve $m(r/(r-1))$ copies. This is on average $<r/(r-1)$ copies per item.

In this case we can say that the amortised cost of appends is $O(1)$ per operation. Insert and delete are still $\varTheta(n)$.

An extensible array will often also shrink when most of it is unused, but this is dependent on implementation.

## Linked lists
Linked lists have a worst case get and set of $\varTheta(n)$ time, inserting a value at the front takes $\varTheta(1)$ time, and insert elsewhere and delete have $\varTheta(n)$ worst case time. 

# Stacks and queues
Sometimes a we know that a list will only be used in specific ways.
- A stack has elements that are only ever added, read, or removed from the front of the list.
	- A linked list is a good implementation for a stack, as adding, removing, and viewing the front of the list is always $\varTheta(1)$
- A queue has elements added to the end of the list and read/removed from the front.
	- A queue can be implemented by using a linked list with a reference to the first and last cells. This means that adding, reading, and removing elements are all $\varTheta(1)$
