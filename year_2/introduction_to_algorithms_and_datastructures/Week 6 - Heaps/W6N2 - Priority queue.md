A priority queue is a [[W4N1 - Classic datatypes#Stacks and queues|queue]], where each element has a priority and the element with the highest priority is always at the front of the queue.

A priority queue has the operations:
- `insertItem(k,e)` insert `e` with priority `k`
- `maxElement()`  return the maximum priority element, error if queue is empty
- `removeMax()` is `maxElement()` except the returned element is then removed from the queue
- `isEmpty()` returns `true` if the queue is empty and `false` otherwise

A priority queue can be implemented using a binary search tree (like a red-black tree) which has all operations in $\Theta(\text{lg}(n))$ (other than `isEmpty()`).
Using a [[W6N1 - Heap datastructure|max heap]] we can get `maxElement()` to $\Theta(1)$ while keeping `insertItem()` and `removeMax()` in $\Theta(\text{lg}(n))$. However a balanced search tree can be tweaked to maintain a direct pointer to the rightmost leaf to give $\Theta(1)$ for `maxElement()`