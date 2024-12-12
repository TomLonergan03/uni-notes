> [!abstract] Definition
>The **network core** is the mesh of interconnected routers that forwards packets from a source to a destination

# Network core functions
There are 2 key functions of the network core:
1. **Forwarding:** move packets arriving to a router's input link to the appropriate output link (a local action)
2. **Routing:** determine the source-destination paths taken by packets using routing algorithms (a global action)
# Packet switching
When an application wishes to send a message, the network stack breaks that message into packets. These packets enter the network through the [[W1N1 - Internet protocol#Network edge|edge]], and routers within the core then forward each packet individually towards its destination.
## Store and forward
Store and forward involves each router storing an entire packet, then forwarding that packet on once it is fully received.
This means that if we send an $L$ bit packet into a link at a transmission rate of $R$ bits per second the overall one-hop end-to-end delay is $2L/R$ (assuming no propagation delay).
e.g. if we wish to send a 10 Kbit packet over a 100 Mbps link it will take $2\cdot\frac{10*10^3}{100*10^6}=0.1\text{ msec}$ .
## Statistical multiplexing
Generally, many devices will want to send packets through the same router. To do this, routers use statistical multiplexing, where the routing allocates a proportion of its total link throughput to each device.
## Queuing delay and loss
If the arrival rate (in bps) to a link exceeds the transmission rate of a link for a period of time packets are queued in a buffer within the router. If the buffer fills up then future packets will be dropped until there is space in the buffer.
## Circuit switching
Instead of switching packets, we can create a full circuit between the start and end point which is reserved for one connection. This guarantees circuit-like performance, but requires many lines on high traffic routes. It was commonly used for landline telephones.
![[w2n1circuitSwitching.png]]
There are 2 ways to increase the number of users of a circuit:
### Frequency division multiplexing (FDM)
For FDM, optical or electromagnetic frequencies are divided into frequency bands. Each call is allocated 1 band on 1 circuit, and can transmit at the max rate of that band.
![[w2n1fdm.png]]
### Time division multiplexing (TDM)
In TDM, time is divided into slots. Each call is allocated a periodic slot, during which it can transmit at the max rate of the wider frequency band.
![[w2n1tdm.png]]
## Packet switching vs circuit switching
Packet switching generally allows more users to use a network, at the cost of more complexity. 
E.g. if we have a 1 Gbps link with users who have 100 Mbps use each when they are active but are only active 10% of the time, circuit switching allows only 10 users to maintain a connection (assuming that they very frequently change between active and inactive), while even with 35 users a packet switched approach would have $<0.0004$ probability of $>10$ users being active at the same time.

Packet switching is good for situations where data comes in bursts, where it is more efficient as it shares resources and doesn't require an initial setup to claim a circuit. It is vulnerable to excessive congestion, where many devices connect but none can get meaningful data through the connection, though this can be limited using congestion control algorithms.
