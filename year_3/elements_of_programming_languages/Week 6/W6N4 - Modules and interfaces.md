# Abstract data types
An **abstract data type (ADT)** is a type together with some operations on it. An ADT abstracts away the type definition and operation implementations are not visible to the rest of the program. Only the types of the operations (the interface) are visible, along with a *specification* which describes its behaviour. Examples of ADTs include lists, stacks, queues, sets, maps, etc.

## Example: priority queue
```scala
object PQueue {
	type T = ...
	val empty: T
	def insert(n: Int, pq: T): T
	def remove(pq: T): (Int, T)
}
```
(this is the equivalent of an *interface*, however Scala does not have an explicit `interface` keyword)

Specification:
- A priority queue (pq) represents a set of integers
- `empty` is the empty set
- `insert` adds to the set
- `remove` removes the *least* element of the set

### One implementation using sorted lists
```scala
object ListPQueue {
	type T = List[Int]
	val empty: T = Nil
	def insert(n: Int, pq: T): T = pq match {
		case Nil => List(n)
		case x::xs => if (n < x) then {n::pq} else {x::insert(n, xs)}
	}
	def remove(pq: T) = pq match {
		case x::xs => (x, xs) // otherwise error
	}
}
```

## Importing
Like packages, `objects` provide a form of namespace:
```scala
val pq = ListPQueue.insert(1, ListPQueue.empty)
import ListPQueue._
val pq2 = remove(pq)
```

In Scala, importing can be done inside other scopes (unlike in Java):
```scala
def singleton(x: Int) {
	import ListPQueue._
	insert(x, empty)
}
```
# Maintaining invariants
On its own, `ListPQueue` is not abstract. If we only use the `ListPQueue` operations then the specification is satisfied, however if we pass in an unsorted list to `remove` (`remove(List(2, 1))`) then `remove` will not return the correct value. This violates the implicit invariant that `T` is a sorted list.

## Private components
As in Java, Scala allows us to make components private:
```scala
object ListPQueue {
	private type T = List[Int]
}
```
This doesn't actually solve the problem, as we can still pass a `List[Int]` in to `remove`.
## Interfaces
We'd like to use an interface `PQueue` that says there is some type `T` with operations:
```scala
empty: T
insert: (Int, T) => T
remove: T => (Int,T)
```
but also prevent clients from knowing the definition of `T`.

### Traits in Scala
```scala
trait PQueue {
	type T = List[Int]
	val empty: T
	def insert(n: Int, pq: T): T
	def remove(pq: T): (Int, T)
}
```
The trait hides information about the implementations of the operations, but we can go further and hide the definition of `T`:
```scala
type T // abstract!
```
Now, we specify that `ListPQueue` implements `PQueue` using the `extends` keyword:
```scala
object ListPQueue extends PQueue {...}
```

## Checking a  module against an interface
For the above trait, and implementation needs to define:
- a type $\tau$ for `T`
- a value `empty` of type $\tau$
- functions `insert` and `remove` with the corresponding types
If it doesn't, then error

## Multiple interface implementations
We can now provide other implementations of `PQueue`:
```scala
object ListPQueue extends PQueue {...}
object HeapPQueue extends PQueue {...}
```
also, in Scala, `object`s can be passed as values, and `extends` implies a [[W5N3 - Subtyping|subtyping]] relationship.

We can define a function that takes any implementation of `PQueue` and creates a new list:
```scala
def make(m: PQueue) = m.insert(42, m.insert(17, m.empty))

val a = make(ListPQueue); // priority queue based on a sorted list
val b = make(HeapPQueue); // priority queue based on a min heap
```

Now, `T` is abstract to clients who use the `PQueue` interface (although not if they were to directly use `ListPQueue`).