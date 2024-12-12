Consensus is required in many parts of distributed systems, such as:
- Replication: replicated data should be the same on all nodes
- Failure detection: nodes must agree on which nodes are functioning correctly or have a fault
- Leader elections: selecting one node to act as a leader, such as to initiate the [[W4N1 - Coordination#Chandy-Lamport snapshot algorithm|Chandy-Lamport snapshot algorithm]]

All of the above scenarios involve multiple parties, potential [[W3N1 - Fault tolerance|faults]], and require a decision to be arrived at without a central service dictating it.
# Consensus protocols
Generally, consensus protocols have some similarities.
For a distributed system with $n$ nodes, each with an input $x_i$, where faults may happen at arbitrary times, we want to agree on a single value, and the value cannot change later. We want to guarantee the following:
- **Termination**: every non-faulty node eventually decides
- **Agreement**: all non-faulty nodes decide on the same value
- **Validity**: the decided value must be the input of at least one node

Consensus protocols are not necessarily democratic, and an input may be selected based on a minority of nodes depending on the protocol.

Consensus is solvable in [[W3N1 - Fault tolerance#System models|synchronous systems]], but is more challenging in asynchronous systems, and all algorithms for asynchronous systems will also work for synchronous systems.
# Impossibility in asynchronous systems
[Fisher, Lynch, and Paterson](https://dl.acm.org/doi/pdf/10.1145/3149.214121) showed that it is impossible to achieve consensus with a single faulty process. They proved that no asynchronous algorithm for agreeing on a one-bit value can guarantee it will terminate in the presence of crash faults.
# Paxos algorithm
**Paxos** (invented by Leslie Lamport) is the most popular consensus solving algorithm, though it does not completely solve the problem. It provides safety guarantees (consensus is not violated) and but does not guarantee liveness (there is a good chance consensus is reached sometime in the future, but termination is not guaranteed).

Nodes can have some combination of these roles:
- Proposers: propose values
- Acceptors: accept proposed values
- Learners: learn proposed value after a consensus is reached

Other assumptions:
- Nodes communicate via messages
- Nodes operate independently and at different speeds
- Nodes can crash or restart while operating
- Message receipt is asynchronous and can be delayed, duplicated, or lost, but never corrupted

For a majority, the system needs $2m+1$ nodes to handle $m$ failures.
## Naive solutions
There are a few simple solutions, that all have issues:
- Single acceptor: $n$ proposers, 1 acceptor
	- The acceptor accepts the first value received
	- This has a single point of failure, so no liveness
- Multiple acceptors: $n$ proposers, $n$ acceptors
	- If acceptors accept the first value they receive, it can result in a split vote
	- If acceptors accept all values they receive, it can result in conflicting choices.
## Proposal numbers and rounds
Each proposal has a unique number, with higher numbers taking priority over lower numbers. A proposer always proposes with a proposal number higher than any it has seen/used. This means that older proposals are rejected in favour of newer ones. A simple approach is $\text{proposal number}=\text{round number}+\text{node ID}$, where the round number is one more than the largest round number seen so far. The round number cannot be reused after crashes or reboots.
## Phases
There are 2 phases:
- Prepare, where we find out any chosen values so far and block older and uncompleted proposals
- Accept, where acceptors are informed to accept a specific value
### Prepare phase
The proposer will choose a proposal number $n$, and send `<prepare, n>` to acceptors. The acceptors on receiving a prepare message, will do the following:
If $n>n_h$, where $n_h$ is the highest proposal seen so far by the acceptor, set $n_h=n$ and reply with `<promise, n, NULL>` if this is the first accepted proposal, or `<promise, n, (n_a, v_a)>` if it has accepted an older proposal.
### Accept phase
If the proposer receives a promise from a majority of the acceptors, it determines any earlier chosen values $v_a$ for $n_a$ and chooses any new or previous value $v$, and sends `<accept, n, v>` to the acceptors.

If $n\geq n_h$, an acceptor will set $n_a=n_h=n$ and $v_a=v$, and reply `<accept, n_h>`.

When the proposer receives a majority of responses, if any $n_h>n$ it starts over from the prepare phase, otherwise the value is chosen.
## Failure handling
If there is one proposer and:
- one or more acceptors fail, the system still works as long as a majority of nodes are up
- the proposer fails in prepare phase, then nothing happens. Another node can propose, or wait for the original proposer to recover
- the proposer fails in accept phase, then another proposal will overwrite the incomplete proposal

If there are two or more simultaneous proposers then it is more complex, and can result in livelocks.
# Multipaxos
Basic Paxos only has two rounds, but for many systems such as databases every single operation would need to go through both basic Paxos rounds, which is costly. Multipaxos reduces the cost by assuming the proposer is stable, then phase 1 elects a proposer, and the accept phase can be run many times with multiple values being accepted.
# Raft protocol
![[w5n1raft.png]]
The Raft protocol is a leader election protocol, which is equivalent to Paxos in fault-tolerance and performance, but is optimised for log appends. Raft elects leaders for fixed length terms, which are followed by another election.
A node can be either:
- A follower, which issue no requests and respond to requests from leaders and candidates
- A candidate, which is used to elect a new leader, and transitions from a follower then transitions into a leader or a follower
- A leader, which handles all client requests

Candidates request votes from followers, who respond yes to the first candidate to request a vote, and whichever candidate gets the majority wins. In the case of a split vote, nothing happens, and a new election is called after the term ends.
## Log replication
The leader's log is the ultimate truth, and while it is leading it ensures that it has all committed entries, and will instruct followers to append new entries and remove uncommitted ones.
# Bully algorithm
Each node has an ID, and the node with the highest ID wins. If we have $n$ nodes $\{N_0,N_1,...,N_n\}$, and whenever a node $N_k$ notices that the leader is unresponsive it initiates an election by sending an election message to all nodes with higher IDs. If  no one responds, $N_k$ wins, but if one or more of the higher ups answer they take over, and send election messages to all nodes with higher IDs.
![[w5n1bullyAlgorithm.png]]
# Ring algorithm
Nodes are organised into a ring, and the node with the highest ID wins. When a node $N_k$ notices the leader is unresponsive it initiates an election by sending an election to the first node after it that is up. When a message is passed on, the node adds itself to the list, and when the message gets back to the initiator it contains all active nodes. This coordinator message is then sent around the ring, and the highest ID in it is elected leader.
![[w5n1ringAlgorithm.png]]