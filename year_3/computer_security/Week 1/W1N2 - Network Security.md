# Stack of layers
Network communication models use a stack of layers.
Higher layers use services of lower layers, with the physical channel on the bottom layer.
A channel is open between 2 devices on every layer, though every layer but the bottom is an abstraction.

# Encapsulation
As a packet is sent and travels down the layers each layer adds its own header and footer.
![[w1n2packetencapsulation.png]]
Each layer only needs to use information in its own header/footer

# Network interfaces
A network interface is a device which connects a computer to a network. e.g. Ethernet card/WiFi adaptor/DSL modem
A computer can have many network interfaces, and each could be doing different things. E.g. a WiFi adaptor for internet access
# Media Access Control (MAC) Addresses
A MAC address controls routing within a LAN.
Most network interfaces come with a predefined MAC address. (Some let you change it)

A MAC address is a 48 bit number typically represented in hex. E.g. 00-1A-92-D4-BF-86.
The first 3 octets are IEEE assigned Organisationally Unique Identifiers, e.g. all Cisco MAC addresses start 00-1A-A1, all D-Link 00-1B-11, ASUSTek 00-1A-92.
The next 3 octets can be assigned by organisations as they wish, but must be unique to each device.

# Switch
A switch performs routing in a local area network
- Operates at the link layer
- Multiple interfaces, each connected to a computer or another switch/router
- A switch will forward each incoming frame to its target device, a hub forwards it to all devices connected
- On connection a computer will broadcast its MAC address to inform the switch what happens

# Internet Protocol (IP)
## Addressing
In order to deliver data IP must be aware of where to deliver data to

## Routing
IP might be required to communicate across networks and communicate with networks not directly connected to the current network

## Fragmentation and reassembly
It is possible that a packet may be carried across a network with a shorter max packet length, so IP must be able to split and combine packets

## IP addresses
An address is subdivided into a <span style="color:green">network,</span> <span style="color:orange">subnet,</span> and <span style="color:red">host</span>
<span style="color:green">128.148.</span><span style="color:orange">32.</span><span style="color:red">110</span>

Broadcast addresses such as 128.148.32.<span style="color:red">255</span> send a packet to all hosts on the network

Private networks are not routed outside of a LAN
- 10.0.0.0/8
- 172.16.0.0/12
- 192.168.0.0/16

## IP header
Includes:
- Source address
- Destination address
- Packet length (up to 64KB)
- Time to live (up to 255)
	- Number of hops before the packet dies
- IP protocol version
- Fragmentation information

![[w1n2IpHeader.png]]

## IP Routing
- A router bridges 2 or more networks
	- Operates at the network layer
	- Maintains tables to forward packets to the appropriate network
	- Forwarding decisions are based solely on the destination address
- A routing table maps ranges of addresses to LANs or other routers

![[w1n2RoutingExample.png]]
Here, a request to server A is directly connected (probably via a switch). As server B is not in the network, the request is passed to the gateway router which then redirects it to whichever router in it's routing table is associated with server B's address, and so on until it reaches server B's gateway, at which point it is sent to server B (again probably via a switch).
## Exploring internet routes
- Internet Control Message Protocol
	- Used for network testing and debugging
	- Simple messages in a single IP packets
	- Is a network layer protocol
- ICMP tools
	- Ping: sends a series of echo request messages and provides stats on roundtrip times and packetloss
	- Traceroute: sends a series of echo requests with increasing TTL
		- This shows what route the packets are taking

# Network attacks
![[w1n2networkingattacks.png]]A Denial of Service (DOS) blocks packets from reaching their destination.
Wiretapping/sniffing takes a copy of each packet which is sent.
Passive wiretapping receives then retransmits each packet unmodified.
Tampering actively alters a packet in flight.
Spoofing results in the destination receiving packets that appear to be from the source but were created by a third party.


