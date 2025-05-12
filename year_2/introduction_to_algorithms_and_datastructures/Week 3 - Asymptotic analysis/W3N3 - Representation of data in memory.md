Program memory is divided into the **stack** and the **heap**.

A value on the stack has a fixed size, and is arranged contiguously. The stack grows and shrinks as variables move in and out of scope. A stack item typically contains either a basic value, such as a number or boolean, or a reference to something on the heap.

Heap objects can be anywhere in memory, be any size, and may reference other objects in the heap. When a heap object is created, the memory manager allocates a region of memory to the object. A heap object may be moved from the original location in some languages, which can result in null references. As the program is executed some heap objects may become unreachable, and some languages use a garbage collector to clean them up, although not all languages do this. In C, for example, this is the cause of memory leaks.

A heap object will often only be copied if it is explicitly requested, otherwise only the reference is copied to a new variable.

We can test for equality either by reference, where the addresses are compared, or by content, where the values at those addresses are compared.

We generally assume that the following operations happen in constant time (i.e. $\varTheta(1)$):
- Reading and writing program variables
- Accessing any field in an object
- Accessing any element in an array
- Allocating a new object on the heap
	- but not initialising its fields
- Allocating a new array on the heap
	- but not initialising its fields

A linked list however, cannot have any value accessed immediately. Finding the $n$th element of a linked list takes $\varTheta(n)$ time.