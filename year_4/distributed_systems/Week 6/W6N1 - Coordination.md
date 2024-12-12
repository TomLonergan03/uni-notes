See [[W9N1 - Synchronisation|here]] for an overview of synchronisation primitives, and [[W10N3 - Deadlocks|here]] for an overview of deadlocks.
# Centralised distributed locks
A simple centralised system for locks [[W5N1 - Consensus|elects a leader]], which then grants exclusive access to one node at a time in the order they request it.
![[w6n1distributedCentralisedLock.png]]
# Ricart-Agarwala algorithm
The **Ricart-Agarwala algorithm** allows for a decentralised approach to locking using logical timestamps. It works as follows:
1. The requester broadcasts a message to all receivers `<resource name, node name, logical timestamp>`
2. For each receiver
	- If it is not accessing the resource and doesn't want to access it, it responds with an `<ok>` message.
	- If it currently has access to the resource, queue the request
	- If it wants to access the resource, compare the timestamp with its local time
		- If the incoming message has the lower timestamp respond with `<ok>`
		- Otherwise, queue the request
3. The requester waits for all nodes to respond with `<ok>`
4. The requester accesses the resource
5. The requester releases the resource and sends `<ok>` to all of its queue entries

![[w6n1ricartAgrawala.png]]

This has the problem that if any node goes down, the resource will not be accessible until the node is restarted.
# Token ring algorithm
![[w6n1tokenRing.png]]
All nodes are arranged in a ring, and a token is passed round in order. If a node has the token, it can access the resource, and if it doesn't need to access the resource it passes the token on to its neighbour. If the token is lost, then consensus must be achieved that it is lost, then the token be recreated.
# Preventing deadlocks
Deadlocks can only occur when all these 4 conditions hold:
1. **Mutual exclusion**: only one node can hold a resource
2. **Hold-and-wait**: a node can hold a lock while waiting to claim another lock
3. **No preemption**: a lock cannot be taken away from a node, only voluntarily released
4. **Circular wait**: a node is waiting for something held by another node, which is waiting for another, and so on in a circle back to the first node

Deadlocks can be prevented by removing any one of the conditions.
## Hold-and-wait
Hold-and-wait can be eliminated by making lock acquisition atomic, with a meta lock that must be held while any other lock is claimed. This requires knowing all locks needed at the same time, claiming all locks that may possibly be needed, and degenerates to one big lock which reduces parallelism.
```c
lock(&meta);
lock(&L1);
lock(&L2);
lock(&L3);
unlock(&meta);
...
unlock(&L1);
...
Unlock(&L2);
...
Unlock(&L3);
```
## No preemption
No preemption can be eliminated by introducing preemption. If a thread can't get what it wants, it will release everything it holds.
```c
top:
	lock(A);
	if (trylock(B) == -1) {
		unlock(A);
		goto top;
	}
```
It is possible for a livelock to occur, if one thread attempts to claim lock `A` then `B` and another attempts to claim `B` then `A`. One solution to this is using exponential random back-off.

Another solution here is to have locks timeout after a period, but selecting the period and determining what happens when multiple locks are held can be difficult.
## Circular waits
Deadlocks can be detected by finding cycles in the wait graphs. This is done by taking snapshots with a [[W4N1 - Coordination#Global snapshots|global snapshot algorithm]], detecting cycles in the waits, and aborting tasks to break the cycle.