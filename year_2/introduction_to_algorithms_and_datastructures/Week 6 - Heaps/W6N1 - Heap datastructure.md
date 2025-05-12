A heap is a binary tree in which every node is either greater than its children (max heap) or less than its children (min heap). The tree is "nearly complete", in that it only has leaves on the bottom and second bottom layer, with all bottom layer leaves grouped to the left.

A heap has the operations:
- Heapify: create a heap from an array $O(n)$
- Insert: insert a node into an existing heap in $O(\text{log }n)$
- ExtractMax: return and delete the top element, then reorder the heap to remain consistent in $O(\text{log }n)$
- Peek: return the top element in $O(1)$

All these examples will be of a max heap, however a min heap just inverts the requirements for swapping

Heapify can be implemented by creating a heap that contains only the first element, then repeatedly calling insert for the remaining elements in the array.

Insert creates a new node in the leftmost missing leaf, then swaps it with its parents until it is less than its parent.

ExtractMax removes the root node, then replaces the root with the last element in the heap, then swaps it if it is less than its children with whichever child is larger until it is greater than it is children.

Peek simply returns the root