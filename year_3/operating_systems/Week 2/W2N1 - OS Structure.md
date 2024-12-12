When designing an operating system, first we must define goals and specifications:
- **User goals:** convenient to use, easy to learn, reliable, safe, fast
- **System goals:** easy to design, implement, and maintain, as well as flexible, error-free, reliable, efficient
It is important to separate policy (what will be done), and mechanism (how that is done).
# Monolithic OS
A **monolithic OS** has the entire operating system running within the kernel.
![[w2n1monolithickernel.png]]
This has the major advantage that subsystem interactions have a low cost as there is no overhead from switching between components, however monolithic OSes tend to be hard to understand, modify, and maintain.
# Layered OS
A **layered OS** creates a series of layers, with each presenting an enhanced "virtual machine" to the layer above, e.g.

| Layer | Name            | Task                                            |
| ----- | --------------- | ----------------------------------------------- |
| 5     | Job managers    | Executes users programs                         |
| 4     | Device manages  | Handles devices and provides buffering          |
| 3     | Console manager | Implements virtual consoles                     |
| 2     | Page manager    | Implements virtual memories for each process    |
| 1     | Kernel          | Implements a virtual processor for each process |
| 0     | Hardware        |                                                 |
This means that every layer can be tested and verified independently, but it also has many problems:
1. It imposes a hierarchical structure which doesn't hold in reality
	- e.g. file the file system requires virtual memory services, but virtual memory would also like to use files for its backing store
2. Crossing each layer has a cost, leading to poorer performance
3. An actual implementation generally ends up not properly following the model
No widely used OS is purely layered, but there are a few layers that are used in modern OSes, mainly a **hardware abstraction layer (HAL)** which separates hardware specific routines from the rest of the kernel. This makes it easier to have a portable OS, as new hardware only requires rewriting the HAL instead of larger changes.
# Microkernel OS
A **microkernel OS** minimises the amount of functionality within the kernel, instead using user-level processes. This results in better reliability as components are isolated from each other, easier extension and customisation, but comes at the cost of poor performance as the user/kernel boundary is crossed much more often.
Microkernels were popular in the 80s and 90s, and have recently had a resurgence.
![[w2n1microkernelStructure.png]]
![[w2n1minix.png]]
# Kernel functionality extension
New functionality can be added to a kernel at compile time, by compiling in more components, or at execution time. This can be done by:
1. Loadable kernel modules ^dd02e9
	- Microkernels load the code in user space
	- Monolithic kernels load the code in kernel space
2. Program injection ^5df907
	- It is possible to load small amounts of code into the kernel on behalf of a process, e.g. WebAssembly

![[w2n1kernelModules.png]]
Loadable kernel modules allow a monolithic OS to gain many of the benefits of a microkernel without the costs:
- It is convenient to add new functionality to the kernel
- It is efficient, as a loaded kernel module doesn't need to cross the user/kernel boundary every time it needs to use another kernel function
- It is flexible, as any module can call any other module unlike in a layered model
