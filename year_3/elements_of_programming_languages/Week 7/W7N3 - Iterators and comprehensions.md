Java provides standard interfaces for *iterators* and *collections*:
```java
interface Iterator<E> {
	public boolean hasNext();
	public E next();
	...
}
interface Collection<E> {
	public Iterator<E> iterator();
	...
}
```
This allows us to work with different collections in a more abstract way than a simple indexed for loop.

For example, you can write:
```java
for (Element x : coll) {
	...
}
```
if `coll` implements `Collection<Element>`.

This is essentially syntactic sugar for:
```java
for (Iterator<Element> i = coll.iterator(); i.hasNext(); ) {
	Element x = i.next();
	...
}
```

In Scala, we have `(x <- coll) { e }` which is syntactic sugar for:
```scala
coll.foreach{x => e}
```
Note that Scala expands `for` loops *before* checking that `coll` actually provides a `foreach` method of appropriate type, and if not, it gives a somewhat mysterious error message:
```scala
scala> for (x <- 42) {println(x)}
<console>:11: error: value foreach is not a member of Int
```
# Comprehensions
Scala, like Haskell, Python, C#, F#, etc. supports a rich "comprehension syntax", e.g:
```scala
scala> for (x <- List("a", "b", "c")) yield (x + "z")
res0: List[Int] = List("az", "bz", "cz")
```
this is a shorthand for:
```scala
List("a", "b", "c").map{x => x + "z"}
```
