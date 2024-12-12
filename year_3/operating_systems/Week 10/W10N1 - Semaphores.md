[[W9N1 - Synchronisation#Locks|Busy waits]] have many disadvantages:
- CPU time is wasted as a thread waits
- High priority jobs can delay themselves if a low priority job holds a lock on a resource the high priority proc needs as it takes longer for that low priority job to be scheduled again and release its lock
- They can only protect one resource at a time
# Sleep and wakeup
Instead of busy waiting, a process could instead call a `sleep()` function, then be woken by the OS when a later thread calls a `wakeup()` function.
# Semaphores
A semaphore is a single variable which indicates whether a critical section is currently occupied, along with a list of sleeping threads. It works like this:
```C
typedef struct {
    int value;
    struct thread *list;
    lock_t lock;
} semaphore_t;

void wait(semaphore_t *S) {
	lock(S->lock);
	S->value--;
	if (S->value < 0) {
		// add the current thread to S->list
		unlock(S->lock);
		sleep();
	} else {
		unlock(S->lock);
	}
}

void signal(semaphore_t *S) {
	lock(S->lock);
	S->value++;
	if (S->value <= 0) {
		// remove a thread T from S->list
		wakeup(T);
	}
	unlock(S->lock);
}
```
Notice that we still have to lock the semaphore itself with a spinlock, but only for when it is being updated. The thread when sleeps until the semaphore next is signalled.
## Semaphore usage
Now, we can use this similarly to a lock, where we call `wait()` to acquire the lock and `signal()` to release the lock. A semaphore can be a **binary semaphore**, where the value is initialised and capped to 1 (so 0 means the lock is held), or a **counting semaphore**, where the value is initialised and capped to the number of protected resources, meaning a positive value means resources are still available, while 0 or negative means that they are all in use. 

A **synchronisation semaphore** has the value initialised to 0, with 0 meaning no events are pending and a positive value meaning events are pending, e.g. if we have two processes $P1$ and $P2$ that requires $P1$ to complete a task $S1$ before $P2$ may perform $S2$. We can do:
```C
semaphore_t sync = {.value = 0};

// in P1
doS1();
signal(sync);

// in P2
wait(sync);
doS2();
```
# Disadvantages of semaphores and locks
Semaphores solve all traditional synchronisation problems, but it is easy to make mistakes primarily based on failing to lock a resource or forgetting to unlock it, and especially when there are multiple locks on a resource it is very easy to deadlock a program.