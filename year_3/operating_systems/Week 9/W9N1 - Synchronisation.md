![[w9n1ordering.png]]
Instructions executed by a single thread/process are totally ordered (A<B<C, A'<B' (X<Y means X occurs before Y)). This is not true when multiple [[W4N1 - Threads|threads]] or [[W3N1 - Processes|processes]] [[W3N2 - Interprocess communication#Shared memory|share memory]], so while B<A' we can't guarantee that C<B' or B<C'. Therefore, we need a way to enforce an order of memory accesses.
# Critical regions
A **critical region** is an area of a program where shared data is accessed. Uncoordinated reads or writes in critical regions can lead to race conditions. A common pattern is a read-modify-write of a shared variable in code that can be executed by concurrent or parallel threads.
# Mutual exclusion (Mutexes)
Mutexes guarantee that at most one thread/process is in a critical section at any one time. We must also guarantee that no process running outside of a critical region can block other processes, that a process will not have to wait forever to enter a critical region, and (it is beneficial though generally not required) that the overhead of entering and exiting a critical section is small relative to the work being done within it.
![[w9n1mutualExclusion.png]]
# Locks
There are a number of synchronisation mechanisms, with the simplest being **locks/spinlocks**, objects with the methods:
- `lock()` - obtain the right to enter the critical section
	- Blocks until the lock is acquired
- `unlock()` - give up the right to be in the critical section
	- Immediate
If a `lock()` call isn't followed by an `unlock()` then other processes/threads can block forever, and care must be taken to take and release locks with the right granularity to minimise the overhead while not blocking other threads for too long.
## Implementation
This requires atomic instructions, which are hardware CPU instructions which perform a load-modify-set operation in one single move, with no interruptions. We will assume `test_and_set()` reads a variable, sets it to true, and returns the read value.
```c
struct lock_t {
	int held = 0;
}

void lock(lock_t* lock) {
	while (test_and_set(lock->held)) { };
}

void unlock(lock_t* lock) {
	lock->held = 0;
}
```
While a thread is blocked by a spinlock, it keeps repeatedly executing the `test_and_set` instruction until it gets preempted by the OS, wasting CPU resources. There are more advanced synchronisation mechanisms that avoid this issue, such as [[W10N1 - Semaphores|semaphores]] and  [[W10N2 - Monitors|monitors]].