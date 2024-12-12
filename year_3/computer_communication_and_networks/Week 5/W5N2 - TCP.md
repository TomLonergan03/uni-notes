TCP is the [[W1N1 - Internet protocol|Internet's]] [[W2N4 - Protocol layering|transport layer]] [[W5N1 - Principles of reliable data transfer|reliable]] transport protocol. 
TCP is:
- **connection-oriented**, meaning that before an application process can start sending data to another, the two processes first "handshake" with each other to determine the parameters of the following data transfer.
- **logical**: consisting of common state only in the hosts at either end
- **full-duplex**: a single TCP connection between processes A and B can send data from A to B and B to A
- **point-to-point**: a TCP connection is between a single sender and receiver
# TCP data transfer
When an application passes data to TCP to send, TCP stores it in its **send buffer**, and then can "send that data in segments at its own convenience" (from the TCP specification, RFC 793). TCP then creates a segment from that data that is limited by the **maximum segment size (MSS)**, which is determined so that the TCP segment when encapsulated in an IP datagram can fit within the largest link-layer frame that can be sent by the sending host. Ethernet has a **maximum transmission unit (MTU)** of 1500 bytes, and the TCP+IP headers are usually 40 bytes, so the MSS is typically 1460 bytes. (Note that confusingly the MSS is the maximum body of a segment, not the maximum segment size overall) ^b004b7
# TCP segment structure
![[w5n2tcpSegmentStructure.png]]
- Similarly to [[W4N3 - UDP|UDP]], TCP has a source and destination port for [[W4N2 - Transport layer protocols#Transport layer multiplexing|multi- and demultiplexing]], along with a checksum
- The 32 bit sequence number and acknowledgement numbers are used for ensuring reliable data transfer
- The 16 bit receive window number is for flow control, and represents the number of bytes the receiver is willing to accept
- The optional options field is usually empty, but is used when a sender and receiver negotiate the MSS as a window scaling factor for use in high speed options, or for timestamping
- The 4-bit header length field specifies the length of the header in 32-bit words. The header is usually 20 bytes (header length = 5), but is more when there are options present ^9a869a
- The flag field consists of 8 bits:
	- CWR and ECE are used in congestion notifications
	- URG indicates that there is data in the segment that is marked as urgent by the sending application, and the last byte of that data is indicated using the 16 bit urgent data pointer
	- ACK indicates that the segment is an acknowledgement for a segment that has been successfully received
	- PSH indicates that the receiver should pass the data to the upper layer immediately
	- RST, SYN, and FIN are used for connection setup and teardown
	- (In practice, URG, PSH, and the urgent data pointer are not used)
# Sequence and acknowledgement numbers
TCP views data as an unstructured but ordered stream of bytes. TCP therefore uses the sequence number to indicate the byte-stream number of the first byte in a segment. The acknowledgement number in a segment is the byte-stream number of the first byte in the opposite direction flow that TCP is waiting for. E.g, if host A has received all bytes between 0 and 535 from host B and A is about to send a segment to B, A will set ACK to be 536.
TCP provides **cumulative acknowledgements**, acknowledging only up to the first missing byte. The TCP standard don't impose any rules surrounding whether or not to cache out-of-order segments, but in practice TCP implementations do store them. The initial sequence number is chosen randomly, to minimise the chance that a delayed in flight packet from a previous connection on the same port could show up with a valid sequence number, and to make it harder for attackers to create fake packets.
# Retransmission timers
TCP uses a retransmission timer with a variable timeout period. The period is calculated using a sample RTT based off of one currently transmitted but not yet ACKed segment, resulting in a new value approximately once every RTT. This is used to calculate an estimated RTT, using an **exponential weighted moving average (EWMA)** using the following formula:
$$
\text{EstimatedRTT}=(1-\alpha)\cdot\text{EstimatedRTT}+\alpha\cdot\text{SampleRTT}
$$
The recommended, but not required, value for $\alpha$ is $0.125$.
The variability of the sample RTT is also calculated, using:
$$
\text{DevRTT}=(1-\beta)\cdot\text{DevRTT}+\beta\cdot\left\vert\text{SampleRTT}-\text{EstimatedRTT}\right\vert
$$
If there is little variation of sample RTT values, then $\text{DevRTT}$ will be small, while if there is a lot of variability it will be large.
The recommended value of $\beta$ is $0.25$.

The timeout value is then set using both these values:
$$
\text{TimeoutInterval}=\text{EstimatedRTT}+4\cdot\text{DevRTT}
$$
This means that the timeout stays close to the estimated RTT, but if $\text{DevRTT}$ grows then so does the timeout period.
An initial $\text{TimeoutInterval}$ of 1 second is recommended, and if a timeout occurs then the timeout is doubled until a segment is received, after which the timeout is recalculated using the above equation.
![[w5n2rttSamplesAndEstimates.png]]
This shows the behaviour of sample and estimated RTT.
# Flow control
TCP provides a flow control service to ensure that the recipient's buffer doesn't overflow. Flow control is therefore a speed matching service, which attempts to match the speed of data transmission to the rate at which the receiving application is reading data from the buffer.
TCP achieves this by having the sender maintain a variable called the receive window. Each time the receiver sends a segment, it includes the current amount of space it has in its receive buffer. The sender uses this value to ensure that the number of unacknowledged bytes in transit is never more than the receive window, ensuring that the receiving buffer never overflows. In the case that the receiver's buffer is full and it gives the sender a window size of 0, and the receiver has no segments it wants to send, the sender will continue to send 1 data byte segments to the receiver until the receiving application starts reading from the buffer again and the recipient will start responding with non-zero window sizes.
# Connection management
## Setup
The 3-way handshake occurs as follows:
1. The client sends a TCP segment with no data to the server, with the SYN flag bit set and a randomly chosen initial client sequence number (`client_isn`) in the sequence field. This is a **SYN message**
2. The server responds with a TCP segment with the SYN and ACK bits set and the acknowledgement number set to `client_isn`+1. It also randomly chooses an initial server sequence number (`server_isn`) and sets the sequence field to that. The server also allocates the buffers and variables to hold data and track the connection. This is the **SYNACK message**
3. Once this segment is received, the client allocates the connection variables and buffers, and acknowledges the server's SYNACK message, by putting the value `server_isn`+1 in the acknowledgement number field. This is the **ACK message**
Once these have occurred, the connection is established and data transfer can begin.
## Teardown
Either party in the connection can end it at any point by sending a segment with the FIN bit set. If e.g. the client sends the FIN message, the server will respond with an ACK, then once the server closes the connection it sends its own FIN message, which the client ACKs, and at this point all variables and buffers are deallocated on both hosts.
# State machines
## Client
![[w5n2tcpClientFsm.png]]
## Server
![[w5n2tcpServerFsm.png]]
