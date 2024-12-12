# QUIC Definition
> [!summary] Definition
> The **Quick UDP Internet Connection (QUIC)** is an application-layer protocol on top of UDP.
> It implements **Error and Congestion Control** similar to TCP, and also implements **connection establishment** for reliability, congestion control, authentication, encryption, and state establishment in one RTT

Multiple application-level "streams" can be multiplexed over a single QUIC connection.

## Connection Establishment
Connection establishment can be achieved with a single handshake, compared to the two handshakes required for the comparable TCP + TLS (adding authentication and crypto state)
![[w5n5QuicConnections.jpg]]
## Parallelisms with Streams
The ability to have multiple parallel streams multiplexed over a single QUIC connection means the congestion window for QUIC is shared across all the streams. As such, there is no Head of Line blocking!