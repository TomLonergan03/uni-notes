Processes can be executed concurrently on a single core system:
![[w4n1concurrent.png]]
or in parallel on a multicore/multiprocessor system:
![[w4n1parallel.png]]
or both:
![[w4n1concurrentAndParallel.png]]
If multiple of these processes need to share data, they can do it by [[W3N2 - Interprocess communication#Message passing|message passing]] (which is slow) or [[W3N2 - Interprocess communication#Shared memory|shared memory]] (which is complicated).
We can use **threads** to give multiple processes concurrency and parallelism in a faster and simpler way.
![[w4n1thread.png]]
Here, each process's instruction flow is isolated, have a private stack, and unique set of registers, but both are in the same address space and share the same OS resources.
# Threads and processes
A process can contain many threads, but a thread is bound to only one process. This means that threads, not processes, become the unit of scheduling (depending on implementation), and processes are just containers in which threads execute.
![[w4n1threadingHistory.png]]
# Address spaces with threads
![[w4n1threadingAddressSpace.png]]
Each thread has its own stack pointer pointing to its own stack, and its own program counter pointing to its next instruction
# Thread control block (TCB)
We've already seen that each process has its own [[W3N1 - Processes#The PCB in Linux|PCB]], but now with threads this must be broken into two pieces: a PCB, which holds memory management and other accounting information, and a **thread control block (TCB)**, which holds a thread's program counter, CPU registers, scheduling information, and pending IO information.

Multithreading is useful on a parallel processor, but it is also useful on a uniprocessor as one thread can run while another is IO blocked.