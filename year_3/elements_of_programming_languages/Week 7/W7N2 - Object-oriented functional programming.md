# Nested classes
A nested class has access to the private/protected members of the containing class, so this can be used to expose an interface associated with a specific object:
```Java
class List<A> {
	private A head;
	private List<A> tail;
	class ListIterator<A> implements Iterator<A> {
		// head, tail are accessible in this implementation
	}
}
```
In Scala, classes and objects can be nested arbitrarily:
```scala
// here, B is constructed automatically
class A { object B { val x = 1 } }
val a = new A

// here, C is constructed at launch while D must be explicitly constructed
object C { class D { val x = 1 } }
val d = new C.D

// here E must be explicitly constructed, then F must be constructed from an instance of E
class E { class F { val x = 1 } }
val e = new E
val f = new e.F
```

## Local classes
A local class is a class that is defined within a method:
```scala
def foo(): Int {
	val z = 1
	class X { val x = z + 1}
	return (new X).x
}
```

# Anonymous classes
Given an interface or parent class we can define an anonymous instance without giving it an explicit name. This is sometimes known as an *anonymous local class*.
```scala
abstract class Foo { def foo(): Int }
val foo1 = new Foo { def foo() = 42 }
```
We can also give a local name to the instance (this is useful as `this` may be shadowed):
```scala
val foo2 = new Foo { self =>
	val x = 42
	def foo() = self.x
}
```
# Parameterised types
Types can take [[W6N1 - Parametric polymorphism|parameters]], e.g. `List[A]` has a type parameter `A`. This is related to (but different from) polymorphism:
- A polymorphic function (like `map`) has a type that is parameterised by a given type
- A parameterised type (like `List[_]`) is a type constructor: for every type `T` it constructs a type `List[T]`

In Scala, there are 3 ways to define parameterised types:
- Type abbreviations with multiple parameters:
  ```scala
  type Pair[A, B] = (A, B)
  ```
- In an abstract class definition:
  ```scala
  abstract class List[A]
  case class Cons[A](head: A, tail: List[A]) extends List[A]
  ```
- In a trait definition:
  ```scala
  trait Stack[A] {
	  ...
  }
  ```

The type parameters of a structure are implicitly available to all components of the structure, thus, in the `List[A]` class, `map`, `flatMap`, `filter` are declared as:
```scala
abstract class List[A] {
	...
	def map[B](f: A => B): List[B]
	def filter(p: A => Boolean): List[A]
	def flatMap[B](f: A => List[B]): List[B]
}
```

## Parameterised types and subtyping
By default, a type parameter is invariant.
To indicate it is [[W5N3 - Subtyping#Subtyping for functions|covariant]], we prefix it with +:
```scala
abstract class List[+A]
```
To indicate it is contravariant, it is prefixed with -:
```scala
trait Fun[-A, +B]
```
Scala does still check that these variance annotations make sense.

## Type bounds
A type parameter can be given a subtyping bounds. For example, in an interface `I`:
```scala
type T <: C
```
means that the abstract type member `T` must be a subtype of `C`.
This is checked for any module implementing `I`.
Type parameters on function or class/trait definitions can be bounded also:
```scala
def f[A <: C](...) = ...
class D[A <: C] { ... }
```
Upper bounds `A >: U` are also possible.

# Traits as mixins
A trait is more powerful than just an interface. A trait can provide fields, and also "default" method implementations.
This means that traits can be used for mixin composition, like in [[W7N1 - Objects and classes#Multiple inheritance|multiple inheritance]], though Scala avoids problems such as diamond inheritance patterns by having the most recent definition wins.

```scala
trait FloorWax { def clean(f: Floor) { ... } }

trait TastyDessertTopping {
	val calories = 1000
	def addTo(d: Dessert) { d.addCal(calories) }
}

object Shimmer extends FloorWax with TastyDessertTopping { ... }
```
Here, `Shimmer` is both a `FloorWax` and a `TastyDessertTopping`

# Scala - object oriented and functional?
So far, we have pretended pairs, lambda-abstractions, etc. are primitives in Scala, this is not true as Scala runs on the JVM, which makes assumptions about primitives.
In Scala, `f(x)` is translated to `f.apply(x)`, and `{ x: T => e }` is essentially syntactic sugar for `new Fun { def apply(x: T) = e }`.
