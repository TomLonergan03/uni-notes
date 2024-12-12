Switches operate at the [[W10N1 - Link layer|link layer]], which means that they don't worry about [[W9N1 - Large scale routing|routing]] or [[W7N1 - The network layer|network layer addresses]].
# MAC addresses
Every network adaptor has its own link layer address, but switches do not. A switch operates transparently, meaning that hosts and routers send link layer messages to each other without being aware of any switches present in the network.
![[w11n1macAddresses.png]]
MAC addresses are assigned at manufacture, with no two adaptors having the same MAC address. MAC address space is managed by the IEEE and manufacturers purchase $2^{24}$ addresses at a time for a nominal fee with the IEEE fixing the first 24 bits and the manufacturer using the last 24 bits as they see fit.

When an adaptor sends a message, it inserts the destination MAC address into the header, and when an adaptor receives a message it discards it if the destination address does not match its own. There is also a special broadcast address (`FF-FF-FF-FF-FF-FF` for Ethernet) which is sent to and processed by all adaptors on the LAN.
# Address resolution protocol (ARP)
The **address resolution protocol (ARP)** maps between IP addresses and MAC addresses.
![[w11n1arp.png]]
If host C wishes to send a message to host A, its adaptor must be told what MAC address the link layer frame needs to be sent to. It finds this by querying its ARP module for what MAC address matches the IP address `222.222.222.222`, and ARP will respond with `49-DB-D2-C7-56-2A`. ARP resolves IP addresses to MAC addresses in a similar sense to how [[W3N7 - DNS|DNS]] resolves application layer hostnames into network layer IP addresses, but with the key difference that while DNS must be able to resolve any hostname anywhere on the internet, ARP is only concerned with the IP addresses on its LAN, and will return an error if an IP address elsewhere on the internet is queried.

Each host or router maintains an ARP table with IP address to MAC address mappings, and each entry has a time-to-live value that determines how long that entry remains in the table. When it is queried, if an entry exists the MAC address is simply returned. If an entry doesn't exist, ARP sends a frame to the broadcast address asking "what is the MAC address corresponding to this IP address". Each other host/router checks to see if that is its own IP address, and the one that matches responds by sending a frame to the sender saying "I am that IP, my MAC address is ...". Finally, the requester stores that MAC address in its ARP table and returns the MAC address.
# Ethernet
![[w11n1ethernetFrame.png]]
An Ethernet frame consists of:
- Preamble: this is 7 bytes of `10101010`, enabling all adaptors on the network to synchronise clocks, then a final byte of `10101011` to alert them that the actual transmission is now coming.
- Source and destination MAC addresses, 6 bytes each
- Type: 2 bytes stating what network layer protocol is contained
- Data: 46 to 1500 bytes of a datagram
- CRC: [[W10N1 - Link layer#Cyclic redundancy check|cyclic redundancy check]] for error detection. Regardless of if the CRC fails, the sender is not notified of the frame's arrival at its destination.

Ethernet is connectionless, which means that a frame is simply sent onto the network, with no coordination required with the recipient. This is one of the places where [[W5N2 - TCP|TCP]] is needed to guarantee reliable delivery.
# Link layer switches
A **switch** transparently moves frames from one interface to another based on a **switch table**. A switch table entry consists of a MAC address, interface, and the time the entry was added to the table. Frames are forwarded to another interface based on their destination MAC address, or may be dropped entirely, but if there is no entry for a specific destination address the frame is broadcast on all links, and if a frame arrives matching a rule to forward it to the same interface it arrived on it is dropped. Similarly to [[W7N2 - Routers#Queuing|routers]], switches have buffers on each of their interfaces in case frames temporarily arrive faster than they can be retransmitted.
## Switch table
The switch table is built dynamically and automatically. When a switch is initialised the table is empty, then when a frame arrives at an interface, the sender MAC address is stored as being on that interface, and if no frame is received from a source after a certain period of time (the **ageing time**) the entry is deleted. This means that when a host joins the network and starts sending data the switch knows where to send replies, and means that if a host leaves the network (or moves to another interface) its entry is removed from the table.
## Properties of switches
1. **No collisions**: if a network is set up as a star with a switch in the centre, only one host is ever broadcasting on a full-duplex link, meaning no collisions occur which significantly increase network performance.
2. **Heterogeneous links**: a switch isolates links from each other which means that different links in a LAN can run at different speeds and over different media.
3. **Management**: switches can automatically disconnect faulty adaptors that send continual nonsense frames, and can collect usage data to allow a sysadmin to monitor and resolve issues easily.
## Switches vs routers
Switches are:
- (+) Plug and play
- (+) Fast
- (-) Limited to spanning trees with no redundant routes
- (-) Expensive for ARP once the network gets large enough
Routers are:
- (+) Configurable to produce desired traffic patterns
- (+) Supports redundant paths
- (+) Provide firewall protection
- (-) Must be configured even to do simple tasks
- (-) Slower
