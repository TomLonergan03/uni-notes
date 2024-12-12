We will look at a few scenarios where congestion occurs:
# Two senders and a router with infinite buffers
For the simplest cause of congestion, we can have two hosts ($A$ and $B$) on one end of a link (with bandwidth $R$) trying to talk to two hosts ($C$ and $D$) on the other end with a router in the middle. We assume the router has an infinite buffer and no traffic control/reliability is performed. 
![[w5n3twoHostsInfiniteBuffers.png]]
Here, if both $A$ and $B$ are transmitting to $C$ and $D$, no matter how fast they try, each will never be able to get more than $R/2$ bps through the link. In addition, it isn't desirable to be close to $R/2$ either, because as we approach it the delay scales exponentially, and once we pass $R/2$ a packet the queue will grow without bound, resulting in infinite wait times.
![[w5n3twoHostsInfiniteBufferChart.png]]
($\lambda$ = quantity of data)
This is one cause of delay, from the **packet arrival rate nearing link capacity.**
# Two senders and a router with finite buffers
If we now consider the same setup, but now the router has a finite buffer, meaning that exceeding $R/2$ bps results in the excess frames being dropped. We will also use a transport protocol with a reliability guarantee, which means that we must also consider retransmitted segments (we will use $\lambda$ to represent new data, i.e. discounting retransmissions, and $\lambda'$ to represent total data). Performance in this situation is very dependent on the retransmission method: 
- Case $a$, the sender magically determines when there is a space in the router, and only sends a segment when there is a space available. In this case $\lambda'_{in}$ matches $\lambda_{out}$, and can never exceed $R/2$.
- Case $b$, the sender uses a long enough timer that it can be near certain a segment hasn't arrived before retransmitting it. In the graph we assume that at $R/2$ in 50% of sent packets get retransmitted, meaning that the $\lambda_{out}=R/3$ for each recipient.
- Case $c$, the sender uses a shorter timer where packets are frequently retransmitted unnecessarily. We assume every packet is transmitted twice, resulting in a $\lambda_{out}=R/4$ at $\lambda'_{in}=R/2$

![[w5n3twoHostsFiniteBuffersChart.png]]

We see in $b$ a cause cost of congestion: **the sender must retransmit data that is lost due to buffer overflows**, and in $c$: **routers are spending time routing duplicate packets**.
# Four senders, routers with finite buffers, and multihop paths
Here, we have four hosts, each transmitting packets over overlapping two hop paths:
![[w5n3fourHosts.png]]

Here, we see the following performance:
![[w5n3fourHostsChart.png]]
This has two components, the first being that when all connections over a link are sending small quantities of data then increasing that quantity is beneficial, as more data can be sent and there is still little chance of overflows occurring. The second component is that once the combined traffic over that link exceeds the link's capacity, then $\lambda_{out}$ actually decreases. This is due to there being a high chance that the buffer is full before any data arrives over one of the connections, meaning that all of that connection's data is dropped, as well as if the second router is full then the first is "wasting" work forwarding packets to it as they are dropped immediately on arrival. This is another cost of congestion: **when a packet is dropped along a path, all the work done by previous routers is wasted**.
# Approaches to congestion control
We can split congestion control systems into two rough categories based on whether the network layer provides explicit assistance to the transport layer for congestion control purposes:
- **End-to-end congestion control**: the network layer provides no explicit support for congestion control, and the transport layer infers network congestion levels based on observed throughput and latency. [[W5N2 - TCP|TCP]] uses end-to-end congestion control.
- **Network assisted congestion control**: the network layer routers provide feedback on congestion levels, ranging from a single bit showing that congestion is present, through to a maximum bitrate it is able to accept.