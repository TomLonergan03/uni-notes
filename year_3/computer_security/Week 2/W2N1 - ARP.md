# ARP
- Connects the network layer to the data link layer
- Maps IP addresses to MAC addresses
- It has no security support
- Is based on broadcast messages and caches of those messages
- Defined in [RFC 826](https://datatracker.ietf.org/doc/html/rfc826)

# ARP messages
There are two types of ARP broadcasts:
1. A request:
	- Who has \<IP addressC>? Tell \<IP addressA>
2. A response:
	- The machine with \<IP addressC> responds
	- \<IP addressC> is at \<MAC address>

The requesting machine then caches the response

# ARP cache
- The command `arp` displays the ARP table (works on Linux, Windows, and OSX)
  ![[w2n1ArpTable.png]]
- An ARP cache entry is stored for a configurable length of time

# ARP cache poisoning
## The attack
- The ARP table is updated whenever an ARP response is received
- It is not tracked whether a ARP response is actually responding to a request
- ARP announcements are not authenticated
This means that a rogue machine can claim to be any other machine on a LAN

This is the normal operation of a LAN:
![[w2n1ArpNormalOperation.png]]

By transmitting fraudulent ARP responses Eve is able to eavesdrop on communication within the network:
![[w2n1ArpPoisoning.png]]
Eve would also be able to block data from reaching a specific/all hosts.

## Mitigations
- **Static ARP cache entries**
	- By hard-coding a MAC address for each IP address this attack is no longer possible
	- This quickly becomes infeasible as networks get larger