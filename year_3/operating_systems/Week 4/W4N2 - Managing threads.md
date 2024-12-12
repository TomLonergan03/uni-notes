There are 2 ways to manage threads:
# Kernel-level threading
The OS is responsible for creating and managing threads. When a process wishes to create a new thread, it would make a [[W1N3 - Syscalls|syscall]], then the kernel would:
1. Allocate an execution stack within the process address space
2. Create and initialise a [[W4N1 - Threads#Thread control block (TCB)|TCB]]
3. Add it to the [[W3N1 - Processes#State queues|ready queue]]

This is **kernel-level threading**, or **1:1** threading, and as a part of it there is a "thread namespace" with a thread identifier (TID) similar to a [[W3N1 - Processes#^956d06|PID]], and a TCB,  similar to the [[W3N1 - Processes#The PCB in Linux|PCB]].
## Advantages
Kernel-level threading allows the OS to schedule all threads, and it can schedule different threads as one blocks, making it possible to overlap IO and computation within a process. Kernel managed threads are cheaper than processes, as there is less state to allocate, initialise, and load/unload during context switches.
## Disadvantages
Kernel-level threading is expensive for fine-grained use, and is multiple orders of magnitude more expensive than a procedure call, as thread operations are syscalls, which involve context switches and argument checks.
# User-level threading
Threads may also be managed within userspace, using a library within the program. This works as the thread manager doesn't need to manipulate memory spaces, and the threads only really differ in hardware contexts such as the program counter, stack pointer, etc. which can be manipulated by user-level code. The thread package is responsible for multiplexing user-level threads within a process. This is known as **user-level threading**, or **1:N threading**, as the kernel is unaware of the threads' existence. TCBs exist at the user level only. User-level threading operations can be 10-100x faster than kernel-level threading operations.
![[w4n2userLevelThreadingPerformance.png]]
For a (very old) example of a 700MHz Intel Pentium CPU running Linux 2.2.16, creating a process using `fork` takes 251 microseconds, a kernel-level `pthread_create()` call takes 94 microseconds (a 2.5x increase), and a user-level `pthread_create()` call takes 4.5 microseconds (another 20x speedup). While these numbers are out of date, the relative performance is still fairly accurate. Userspace threading doesn't allow for executing multiple threads across several cores on a CPU, which requires the kernel to be involved.
## User-level thread scheduling
There are 2 approaches to ensure CPU time is shared between user-level threads:
1. Every thread willingly cooperates
	- A thread chooses to give up the CPU by calling `yield()`
	- `yield()` calls into the scheduler, which context switches to another ready thread
	- If a thread never calls `yield()`, then it will keep the CPU forever (within that process at least)
2. **Preemption**
	- The scheduler requests a timer interrupt be delivered by the OS periodically (usually as a UNIX signal), and each time the timer interrupt occurs the scheduler gains control and context switches as appropriate
## User-level thread IO
If a user-level thread makes a synchronous IO operation, the kernel treats the *whole process* as being blocked and schedules another process in its place, even if other user-level threads in the original process would still be able to make progress. This is not the case with kernel-level threading, as there the kernel knows about all threads within each process.
# The N:M threading model
The **N:M threading model** has a userspace library which registers several kernel-level threads with the OS, and also manages the user-level threads. This means that the userspace library can perform high speed thread switching, but also that the kernel doesn't block the entire process when one thread is waiting for IO.
![[w4n2nmThreading.png]]
# APIs
For user- or kernel-level threading, the standard API is the POSIX Thread/`pthread` API:
- `ret = pthread_create(&t, attributes, start_procedure);` creates a new thread, which starts executing at `start_procedure`
- `pthread_cond_wait(conditional_variable, mutex)` blocks the calling thread until `conditional_variable` changes
- `pthread_signal(conditional_variable)` resumes a thread waiting on the `conditional_variable`
- `pthread_exit()` terminates the calling thread
- `pthread_join(t)` waits for thread `t` to terminate

There are many other libraries, often offering compiler-level support using code annotation, C preprocessor `pragmas`, or templates, including:
- Thread pools
- Fork-join
- OpenMP
- Grand Central Dispatch
- Intel Thread Building Blocks
