# Parallel vs distributed DBMSes
**Parallel DBMSes** have nodes that are physically close to each other, connected via a high-speed LAN, and low communication costs.

**Distributed DBMSes** can have nodes that are far from each other and connected over a public network, meaning communication cost and problems cannot be ignored.

A **distributed transaction** can access data located on multiple nodes. The DBMS must guarantee the [[W7N1 - Transactions#ACID properties|ACID properties]] despite this.
# Distributed concurrency control
If a DBMS is distributed over a number of nodes, we can partition the locks with the data, so each node manages its own locks. For coarser grain locks (e.g. whole table/database locks) a "home" node can manage them for the whole system.

This still leaves global issues of deadlocks and commits/aborts.
# Distributed deadlock detection
![[w7n3distributedDeadlock.png]]
Even if there is no cycle in the local waits-for graph, there can still be a cycle in the global waits-for graph. This can be fixed by periodically unioning all local graphs at one node, and aborting a transaction to break the cycle.
![[w7n3globalDeadlockDetection.png]]
# Distributed two phase commit
There are a number of things that can go wrong during a transaction:
- A node may crash or become disconnected, then must catch up to the system state.
- Messages may be lost, delayed, or reordered.
- Each node may choose to abort a transaction

This is solved using distributed voting through the **2-phase commit (2PC)** protocol, which has (perhaps obviously) 2 phases:
1. **Voting**: a coordinator tells participants to "prepare". Each participant responds with a yes or no vote, with unanimity required for a transaction to commit.
2. **Commit**: the coordinator distributes the result of the vote.

With [[W7N2 - Recovery#Write ahead logging|logging]], phase 1 goes as:
1. The coordinator tells participants to prepare.
2. Participants generate a prepare/abort record.
3. The participants flush the prepare/abort record to disk.
4. The participants respond with their yes/no votes.
5. The coordinator generates a commit record.
6. The coordinator flushes the commit record to disk.

Then in phase 2:
1. The coordinator broadcasts the result of the vote.
2. Participants make a commit/abort record.
3. Participants flush the record.
4. Participants respond with an acknowledgement.
5. The coordinator generates a transaction end record.
6. The coordinator flushes the record.

![[w7n32phaseCommit.png]]
## Recovery and 2PC
We assume all nodes will eventually recover, otherwise a rollback to a backup is required. This depends on write-ahead logging, and short downtimes.

If the coordinator notices a participant is down, if that participant hasn't voted yet the coordinator aborts the transaction, while if the coordinator is waiting for a commit ack then it hands off to the recovery process.

If the participant notices the coordinator is down, then if it hasn't yet logged prepare the participant aborts the transaction unilaterally, while if it has it is handed off to the recovery process.

The coordinator recovery process gets inquiries from participants in the prepared state, and if the log at the coordinator says abort or commit the appropriate response is sent and the protocol continues, while if the log says nothing an abort is sent.
## 2PC and 2PL
When using [[W7N1 - Locking#Strict 2-phase locking|strict 2 phase locking]], aborts are safe for any node to perform unilaterally as there are no cascading aborts. 2PC requires point-to-point messages to be correctly ordered, which can be achieved by buffering messages until a run of messages is received in order, (e.g. using [[W5N2 - TCP|TCP]]).