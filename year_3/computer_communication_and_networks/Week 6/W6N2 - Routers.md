![[w6n2routerInternals.png]]
A router consists of:
- **Input ports**: physically terminates the incoming physical link, a queue of messages waiting to be routed, and finally performs the lookup to determine where the packet should be routed to
- **Switch fabric**: connects all input ports to all output ports, and is used to move a datagram from an input to an output
- **Output ports**: contains a queue of messages waiting to be transmitted, and the physical transmitter used to send messages onto the next physical link. (Note that sometimes there will be an input and an output port on a physical link, if the physical link supports 2 way communication)
- A **routing processor**: performs control plane functions such as maintaining the routing tables
# Input port forwarding
When a packet arrives, the forwarding table is queried to see where to send it. This is done using **longest prefix matching**, where the table contains a number of prefixes of datagram headers, and whichever is the longest matching entry is used to select the output port.
# Switching
There are several approaches to switching packets from an input port to an output port:
- **Switching via memory**: the oldest approach. The router is simply a computer, and when a packet arrives it triggers an interrupt, then the routing processor copies the packet to memory, determines how to route that packet, then copies it back out to the output buffer. In this scenario, if the processor has a bandwidth of $B$ bps, then the overall throughput must be less than $B/2$. In addition, only one packet can be routed at a time, as only one memory read or write can occur over the shared system bus.
  ![[w6n2memorySwitching.png]]
- **Switching via a bus**: here, the input port prepends an internal routing label to the packet indicating the correct output link. This is then written to a bus connecting all inputs to all outputs, and only the indicated output stores the packet. Here, the switching speed of the router is limited by the bus speed.
  ![[w6n2busSwitching.png]]
- **Switching via an interconnection network**: by connecting a network of busses with toggleable connections at each intersection multiple packets can be routed in parallel. In the example below, a packet arriving at $B$ or $C$ could be routed to outputs $X$ or $Z$
  ![[w6n2interconnectSwitching.png]]
# Queuing
Queuing occurs before switching and before transmission, and packets may be [[W2N3 - Network performance#Packet loss|dropped]] at either.
## Input queuing
Queues build up at the input if the switching rate is slower than the rate of arriving packets. This can be simply due to too many arriving, or in interconnection networks due to **head of line (HOL) blocking**, where a packet has to wait as its output is already in use, causing the packets behind it on a link to be blocked. It has been shown that the input queue will grow without bound as soon as the packet arrival rate on an input link reaches 58% of its capacity. Below, the light blue packet experiences HOL blocking:
![[w6n2holBlocking.png]]
## Output queuing
If packets arriving across multiple input ports that all get switched to the same output port, then a queue can also build up there too.
## Dropping packets
When a queue is full, then either incoming packets or currently queuing packets must be dropped.
# Packet scheduling
Packets can only be transmitted from an output port one at a time, and thusly there must be a system to determine which gets sent next.
## FIFO scheduling
The simplest is **first-in-first-out** scheduling (very similar to the [[W4N4 - Scheduling algorithms#First-come first-served (FCFS)|similarly named process scheduling system]]) where the oldest queuing message is selected for transmission at each time-point.
## Priority scheduling
Here, arriving packets are classified into priority classes giving some packets priority for transmission (e.g. network management packets, or prioritising VoIP over [[W3N6 - E-mail#Simple mail transfer protocol (STMP)|STMP]]). Each priority class then has its own queue, and then the highest priority queue containing a packet is transmitted from using FIFO. This has the risk that if there is enough high priority traffic to saturate the router then no lower priority traffic will ever make it past that router.
## Round robin scheduling
Instead of ranking packets by priority, we can instead alternate between classes, sending $N$ packets before moving to the next class. This can be further developed by allocating a different amount of time to each of the classes, which allows for priorities while also prioritising specific types of packets.