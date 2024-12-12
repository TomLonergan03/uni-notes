If two processes both need exclusive access to two resources, we can run into an issue.
```C
lock_t resource_a, resource_b;

// Process A
{
	// do some stuff
	lock(resource_a);
	lock(resource_b);
	// use both resources
	unlock(resource_a);
	unlock(resource_b);
}

// Process B
{
	// do some stuff
	lock(resource_b);
	lock(resource_a);
	// use both resources
	unlock(resource_b);
	unlock(resource_a);
}
```
Here, process A may claim a lock on `resource_a`, and simultaneously process B claims a lock on `resource_b`. In this case, neither can make progress as both need the other process' claimed resource before it can unlock its own. This is a **deadlock**.
More generally, a deadlock occurs when every process in a set of processes is waiting for an event that can only be caused by another process in that set.
## Livelocks
A livelock occurs when a process is continuously attempting an action that fails. This is generally less serious than a deadlock, as sometimes the action can eventually succeed e.g. when random chance leads to all required resources being available simultaneously.
## Necessary conditions for deadlocks
1. **Mutual exclusion**: a single resource can only be assigned to one process at a time
2. **Hold and wait**: processes currently holding resources can request new resources
3. **No preemption**: resources cannot be taken away after being granted to a thread, and must be explicitly released by that thread
4. **Circular wait**: there is a circular list of two or more processes where each is waiting for a resource held by the next member of the list
# How to avoid deadlocks
There are 4 approaches to dealing with deadlocks:
1. **Deadlock prevention**: use specific programming techniques or language features to stop deadlocks when code is written. This requires effort from the programmer
2. **Deadlock avoidance**: at runtime deny resource requests if they may deadlock
3. **Detect and recover**: at runtime detect deadlocks and recover from them
4. **Do nothing**: hope the programmer is smart enough to avoid them and never makes a mistake. This is the approach used by most operating systems, including Linux and Windows
## Deadlock prevention
We can do this by preventing any one of the necessary conditions:
1. Mutual exclusion can be reduced by using a mediator which takes sole responsibility for accessing an object. This is not perfect though, as processes could still deadlock filling the mediator
2. Hold and wait can be attacked by requiring all processes to claim everything they need in one go, and allocate all resources in one go meaning that if any resource is busy that process will wait. This is difficult, as processes may not know which resources they need to complete their task
3. No preemption can be attacked by releasing all held resources when a process attempts to claim another and is blocked. This can only be applied to resources whose state can be saved and restored later ([[W3N1 - Processes|CPU time]], [[W6N2 - Virtual memory|memory space]], etc.) but not all resources can be virtualised
4. Circular wait can be attacked by setting an order in which locks are acquired, and always acquiring all those locks in that order. This leads to resources being locked unnecessarily
## Deadlock avoidance
Here, the OS/runtime is given information in advance on what resources a thread will request and use during its lifetime, as well as the complete sequence of requests and releases of each thread. Then, for each request the runtime decides if that request should wait considering the current resource-allocation state, and what future requests are going to come in.
The core of deadlock avoidance is the **safe state**. A system is in a safe state only if there exists a sequence $<P_1,P_2,...,P_n>$ of all processes in the system such that for each $P_i$, the resources it requests can be satisfied by all currently available resources plus all resources held by all $P_j, j<i$. In this state, a $P_i$ has a resource need that is not immediately available it can wait until all $P_j$ have finished, claim its resources, execute, return them, and terminate, allowing $P_{i+1}$ to claim its resources.
### Banker's algorithm
The Banker's algorithm is an algorithm for deadlock avoidance, which maintains requires each process to list its total number of required resources.
If we have $n$ processes and $m$ resource types, we then need:
- A vector of length $m$ storing the number of available resources of each type
- An $n$ x $m$ matrix containing the maximum demand of each process
- An $n$ x $m$ matrix containing the currently allocated resources to each process
- An $n$ x $m$ matrix containing the remaining resources each process needs, this is calculated using $Need[i,j]=Max[i,j]-Allocation[i,j]$
A process can be granted its requests only when it is able to fully satisfy all its needed resources, otherwise it has to wait.
## Deadlock detection and recovery
Once a deadlock is detected, a system could simply terminate all deadlocked processes, or one at a time until the deadlock cycle is eliminated. This has the drawback of potentially leaving resources in incorrect states. Another approach is to preempt resources from threads and give them to others until the cycle is broken.