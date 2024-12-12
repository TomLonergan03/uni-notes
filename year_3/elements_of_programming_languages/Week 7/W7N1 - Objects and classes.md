# Objects
An object is a [[W6N4 - Modules and interfaces|module]] with some additional properties:
- **encapsulation:** access to an object's components can be limited to the object itself or a subset of objects
- **self-reference:** an object is a value and its methods can refer to the objects fields and methods via an implicit parameter, often called `this` or `self`
- **Inheritance:** an object can inherit behaviour from another "parent" object

In Scala, the `object` keyword creates a singleton object ("class with only one instance")
## Self reference
Inside an object definition, the `this` keyword refers to the object being defined.
```scala
object Fact {
	def fact (n: Int): Int = if (n == 0) {1} else {n * this.fact(n - 1)}
}
```

## Encapsulation and scope
An object can restrict the scope of its members, e.g. `public` and `private` keywords in Java and C++.
Java has package scope by default, which is that a member is visible only to other components in the same package
Scala has `private[X]` for a *qualified scope*, i.e. private to class/object/trait/package `X`
Python does not have an enforced private scope, so must rely on programmer goodwill to not manipulate internal states of objects.

# Classes
A class is an [[W6N4 - Modules and interfaces#Interfaces|interface]] with the additional properties:
- **Instantiation:** classes describe how to construct associated objects (*instances* of the class)
- **Inheritance:** classes may inherit from zero or more parent classes as well as implement zero or more interfaces
- **Abstraction:** classes may be abstract, i.e. may name but not define some fields or methods
- **Dynamic dispatch:** the choice of which method is called is determined by the runtime type of a class instance`

Note that not all object oriented languages have classes, e.g. Smalltalk or JavaScript. Such languages often use prototypes, which are commonly-used objects that play a similar role to classes.

## Constructing instances
Classes typically define special functions that create new instances, called constructors
- In C++/Java, constructors are defined explicitly and separately from initialised data
- In Scala, there is a default constructor whose parameters are in scope in the whole class body, and other constructors can be defined as needed
```scala
class C(x: Int, y: String) {
	val i = x
	val s = y
	def this(x: Int) = this(x, "default")
}
scala> val c1 = new C(1, "abc")
scala> val c2 = new C(1) // this uses the manually defined constructor
```
## Inheritance
An object can inherit from another, meaning the parent object and its components become part of the child object (often accessible using the `super` keyword). In Java and Scala, a class extends exactly one superclass (defaulting to `Object` if none is specified). C++ allows multiple inheritance from several parents.
Non-class-based languages, like JavaScript and Smalltalk, support inheritance directly on objects via extension.
### Subtyping
An object `Obj` that extends a trait `Tr` is automatically a [[W5N3 - Subtyping|subtype]], likewise so is a class that extends a trait or a class/object that extends another class/object.
Subtyping and inheritance are distinct feature; subtyping can exist without inheritance, and subtyping is about types, while inheritance is about behaviour.

### Inheritance and encapsulation
Inheritance makes encapsulation more complex, and generally it means that `private` prevents access from outside the class - even in a child class - while `protected` means it is both accessible within the class and any children.

### Cross instance sharing
Classes in Java can have `static` fields/members that are shared across all instances.
`static` methods can access `private` fields and methods. `static` is also allowed in interfaces.
A class with only static members is equivalent to a module.
C++ has the `friend` keyword which allows sharing between classes on a case-by-case basis.

### Companion objects
Scala doesn't have a `static` keyword, and instead uses *companion objects*, where an object with the same name as a class are companions, and can access each other's private components.
```scala
object Count { private var x = 1 }
class Count { def incr() = {Count.x = Count.x + 1}}
```

### Multiple inheritance
C++ supports multiple inheritance, but here we will use a (non-compiling!) Scala example:
```scala
class Win(val x: Int, val y: Int)
class TextWin(...) extends Win
class GraphicsWin(...) extends Win
class TextGraphicsWin(...) extends TextWin and GraphicsWin
```
In C++, this means there are two copies of `Win` inside `TextGraphicsWin`, which can easily become out of sync. Multiple inheritance is also difficult to implement efficiently, so modern languages tend to avoid it.

### Abstraction
A class may leave some components undefined. Such classes must be marked as `abstract` in Java, C++, and Scala. To instantiate an abstract calls, definitions for the methods must be defined, e.g. in a subclass.
Abstract classes are used to define common behaviour to be inherited by subclasses.
In Scala, abstract classes can also have unknown type components, optionally with subtype constraints.
```scala
abstract class ConstantVal {
	type T <: AnyVal
	val c: T
} // This is a constant of any Value type
```

### Dynamic dispatch
An abstract method can be implemented in different ways by different subclasses, and when that method is called, the corresponding implementation is determined at runtime.
```scala
abstract class A { def foo(): String }
class B extends A { def foo() = "B" }
class C extends A { def foo() = "C" }

scala> val b: A = new B
scala> val b: A = new C
scala> (b.foo(), c.foo()) // evals to ("B", "C")
```
### Overriding
An inherited method that is already defined by a superclass can be overridden in a subclass. This means that the subclass's version is called during dynamic dispatch. 
In Java, the `@Override` annotation is an optional, checked documentation that a method is overriding an inherited method.
In Scala, the `override` keyword must be used to override a function.

### Type tests and coercion
Given `x: A`, Java/Scala allow us to test whether it is actually a subclass `B`:
```scala
scala> b.isInstanceOf[B]
```
and to coerce such a reference to `y: B`:
```scala
scala> val b2: B = b.asInstanceOf[B]
```
This means we can violate [[W6N1 - Parametric polymorphism|type abstraction]], so thought should be given as to when to use it:
```scala
def thisIsBad[A](x: A) = if (x.isInstanceOf[Int]) {
	(x.asInstanceOf[Int] + 1).asInstanceOf[A]
} else {x}
```
