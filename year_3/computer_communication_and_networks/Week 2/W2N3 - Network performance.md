An ideal network would be able to move an unlimited amount of data between any two connected devices, instantaneously and without any data loss. This is (obviously) not possible, as physics introduces delay, loss, and throughput constraints.
# Types of delay
As a packet travels through a network, it suffers from several types of delay at *every* node it encounters.
## Processing delay
When a packet arrives at a node, the header is examined and used to determine where the packet should be directed - this takes some time and is **processing delay**. Processing delays in modern high-speed routers are usually microseconds or less.
## Queuing delay
After a packet is processed, it is added to queue to wait for its transmission link to become available. This is dependent on how many packets have arrived before it that are still waiting to be transmitted, if no packets have arrived recently then it will be close to 0, but if traffic is heavy then there may be a long delay. In practice, queuing delays are between microseconds and milliseconds.
## Transmission delay
A link between routers will have a transmission rate in ($R$ bits per second). If a packet is of size $L$ bits, then it will take $L/R$ seconds to transmit one full packet. Transmission delays tend to be between microseconds and milliseconds in practice.
## Propagation delay
Once a bit is pushed into a link, it must propagate down it to the next router. This is governed by the propagation speed (the speed that the signal can move within the link's medium, e.g. speed of light (~$3*10^8\text{ m/s}$) in fibre lines or the speed of electricity (~$2*10^8\text{ m/s}$) in copper). Propagation delay can be milliseconds in large networks, or nearly negligible in local ones.
## Overall delay
If we let $d_{proc}$, $d_{queue}$, $d_{trans}$, $d_{prop}$ be the processing, queuing, transmission, and propagation delays respectively, then the total nodal delay
$$d_{nodal}=d_{proc}+d_{queue}+d_{trans}+d_{prop}$$
# Queuing delay in more detail
Queuing delay is the most complex part of nodal delay, as it varies packet by packet. This means that we can't discuss absolute values but instead statistical measures of mean/median delays and their distributions, or in probabilities.
If we have $R$ = transmission rate, $a$ = packets per second, and $L$ = packet size, then we can calculate the traffic intensity:
$$
\text{Traffic intensity }=\frac{La}{R}
$$
If $\frac{La}{R}>1$ then the average rate of bits arriving at the queue exceeds the rate at which they can be transmitted, resulting in the queue increasing forever and the queuing delay approaching infinity.
In the case $\frac{La}{R}\leq1$ the nature of arriving traffic impacts the queuing delay. E.g. if one packet arrives exactly every $\frac{L}{R}$ seconds then each packet will arrive to an empty queue and the will be no queuing delay. If the packets arrive in periodic bursts, e.g. $N$ packets every $\frac{NL}{R}$ seconds, then the first packet will have no queuing delay, the second an $\frac{L}{R}$ second delay, third a $\frac{2L}{R}$ delay, and so on with the $n$th packet having a $\frac{(n-1)L}{R}$ delay.

In practice, both of these are very uncommon, with traffic tending to be (nearly) random. In this case, the traffic intensity equation doesn't strictly hold but it is still a convenient way to think about queuing delays. In this case, as the traffic intensity is near 0 there is a low probability of a packet having to queue, and as it gets closer to 1 there will be more and more periods during which it is greater than 1 leading to queues building up and delays increasing.
# Packet loss
So far, we're assuming that a router can hold an unlimited number of packets. In practice, it can't, and instead if a packet arrives to a router with a full queue it is dropped. This means that the packet is lost, and will never arrive at its destination. Therefore, nodal performance is often measured in terms of both delay and probability of packet loss.
# End to end delay
End to end delay is measured simply by summing the nodal delay of every node along the route.
# Other delays
There are other sources of delay, e.g. in a shared medium like WiFi an end system may purposefully delay transmission to enable other devices to share the medium. In some applications, it takes time to gather and packetise the data before it is sent, this can be a source of delay in things like VoIP applications.
# Throughput
The end-to-end throughput of a route is determined by the minimum throughput link within that route. In the modern internet, the [[W2N1 - Network core|core]] has links that are so high speed they are rarely the bottleneck, with it instead being the access link of either the client or server. It is important to note that throughput is directly divided between all traffic using that route, so if there is one link in a route that has very high traffic it can significantly reduce throughput.