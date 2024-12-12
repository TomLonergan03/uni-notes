There are a number of transport services from which an application developer chooses based on that application's needs.
Some needs include:
# Reliable data transfer
There are many ways data can be lost in transit: it could [[W2N3 - Network performance#Packet loss|overflow a buffer and be dropped]], be corrupted in transit and be discarded by the host, or the host could momentarily lose connection as the packet was arriving. A protocol may provide a guarantee that the data that is sent using it will arrive uncorrupted at its destination, and is said to provide **reliable data transfer**.
Some applications don't require all sent data to arrive, e.g. video or voice calls, where lost data leads to a degradation of video/audio quality, and in this case it may not be worth the additional overhead of ensuring reliable transfer.
# Throughput
A transport protocol could provide minimum throughput guarantees. Some applications require a minimum amount of bandwidth, such as video or audio streaming, though they may make use of adaptive encoding techniques to vary the amount needed to attempt to match the connection's throughput, though there is still a minimum bandwidth required. These are known as **bandwidth sensitive applications**. Other applications, such as email or file transfer can make use of however much bandwidth is available (although more is always better), and are known as **elastic applications**.
# Timing
Some applications require a maximum amount of time between a packet being sent and it making it to its destination, such as online multiplayer games. In this case, a transport protocol could provide a maximum latency guarantee. Again, there are many applications where latency doesn't matter very much, but lower latencies are always better.
# Security
A transport protocol could provide security services, such as encrypting data in transit or guaranteeing its [[W3N5 - Message Authentication Codes (MACs)|authenticity or integrity]]. This would allow applications to communicate securely with the guarantee that nobody in the middle had read or altered that communication.
