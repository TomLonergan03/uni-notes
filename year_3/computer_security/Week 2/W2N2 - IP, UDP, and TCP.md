# Vulnerabilities
- Unencrypted transmission
- No source authentication
	- A sender can give whatever source address they want, so it can be hard to trace a packet back to an attacker
	- Allows [[W1N2 - Network Security#Network attacks|spoofing attacks]]
- No integrity checking
	- The entire packet is modifiable, meaning that both the header and payload could be modified between transmission and delivery
	- Enables [[W1N2 - Network Security#Network attacks|tampering attacks]]
- No bandwidth constraints
	- A large number of packets can be broadcast to launch a [[W1N2 - Network Security#Network attacks|denial of service attack]]
	- Broadcast addresses make it easy to DOS many devices simultaneously

# User Datagram Protocol (UDP)
UDP is a stateless, unreliable datagram protocol at the transport layer. It does not provide delivery guarantees or acknowledgements which makes it efficient, but an application using UDP must be able to accept some corrupted or lost data. It is also capable of distinguishing data for multiple concurrent applications on a single host.

Uses of UDP include video/audio streaming and VoIP.

# Transmission Control Protocol (TCP)
TCP is a stateful transport layer protocol for reliable, in-order delivery of messages, with the ability to distinguish multiple applications on the same host.
TCP packages a datastream into segments transported over IP. Order is maintained by adding a sequence number to each packet.
When TCP receives a packet, it sends out an acknowledgement (ACK) to indicate successful reception of the packet. If a sender does not receive the ACK within a timeout period, the packet is retransmitted.
TCP generally checks the data received by comparing a checksum encoded in the packet with one generated off of the contents of the packet.

TCP generally maintains a certain number number of packets in-flight, and will only send more packets once previous ones have been ACKed.

# Ports
TCP & UDP support concurrent applications on the same device.
A port is a 16 bit number which identifies where data is directed.

Ports 0-1023 are reserved for use by known protocols, e.g. HTTPS uses 443 (and sometimes 8080), HTTP uses 80, SSH uses 22

These are not required, however generally applications will assume that e.g. an attempt to SSH into a device will use port 22.

Ports 1024-49151 are user ports, and are used to listen to connections.

The source and destination ports are contained in the TCP header.

# TCP packet format
![[w2n2TcpFormat.png]]

# TCP handshake
TCP is stateful, and in order to establish an initial connection a three way hand shake is used.
First a connection is requested by the client sending a SYN packet to the server. Then, the server responds with a SYN/ACK packet, acknowledging the connection. Finally, the client responds by sending an ACK to the server, establishing the connection
![[w2n2TcpHandshake1.png]]
![[w2n2TcpHandshake2.png]]
TCP keeps track of packet loss, network congestion, and flow control.

# TCP attacks
## SYN Flooding
By sending thousands of SYN packets without acknowledging any replies, a target's state table can run out of space and is unable to establish new connections.

### Issues
- If the attacker uses their own IP they can be easily traced after the fact.
	- A way to get around this is to spoof the source address, however ingress filtering is commonly used to drop packets with source addresses outside of their origin fragment
- An attacker's bandwidth is likely to be smaller than a server's
- Therefore, it is effective against a small target like someone hosting a game server in their home, however it is likely ineffective against a large company website.

## Smurfing
Some networks respond to [[W1N2 - Network Security#Exploring internet routes|pings]] to broadcast addresses. These networks are called "Smurf amplifiers".

Therefore, if we send a forged ping packet with the victim's IP address as the source to a Smurf amplifier, the broadcast address will then send the ping to all computers on the network, which all reply to the victim.

This is significantly more powerful than sending the pings from one computer, as it uses the bandwidth of a large number of computers, and only requires one packet to set it off.