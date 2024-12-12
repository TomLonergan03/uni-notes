[[W9N4 - References and Arrays|References and arrays]] illustrate a common resource pattern, where we allocate/open/acquire a resource, use it, then deallocate/close/release it. This shows up in many places, e.g. memory usage, files and file descriptors, databases, network connections, and locks
# References and types
Some languages - C/C++ notably - distinguish between type $\tau$ and $\tau*$ (a pointer to $\tau$). Other languages, notably Java, consider many types to be reference types, i.e. all variables of that type are mutable and nullable references.

In Scala, variables introduced by `val` are immutable, while using `var` means variables can be reassigned.
In Haskell, as a pure, functional language, all variables are immutable. References and mutable state are possible, but require special handling.
# Allocation of resources
In a strongly typed language, we can ensure safe resource use by ensuring all expressions of type $\text{ref}[\tau]$ are properly initialised.
C and C++ do not do this, a pointer $\tau*$ may be uninitialised, and may require separate initialisation via `malloc` or similar.
Java does this to some extent, as an expression of type $\text{ref}[\tau]$ is a reference to either an allocated $\tau$ or to null.
Scala and Haskell don't allow silent null values, so a $\text{ref}[\tau]$ is always an allocated structure
# Deallocation of resources
Types are not as helpful in enforcing safe deallocation.
Both forgetting to deallocate a resource (memory leaking) and deallocating the same resource multiple times or attempting to use after free will cause problems.
This can occur as copies of references to allocated resources can often propagate to unpredictable parts of the program.
It is possible to avoid this through advanced typing systems, such as affine typing in Rust.
## Main approaches
### Explicit deallocation
C/C++ use explicit deallocation (via `free`) where the programmer manually declares when to release resources. This is hard to get right, and is a major cause of bugs and security vulnerabilities.
### Garbage collection
Java/Scala/Haskell use garbage collection, where a runtime decides when resources are not going to be used again and automatically deallocates them.
## Other approaches
- Reference counting involves keeping track of how many pointers currently exist to a resource
- Rust keeps track of what owns a reference, and deallocates it when that reference goes out of scope