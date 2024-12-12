A **firewall** is a security measure designed to prevent unauthorised electronic access to a networked computer system.

A firewall intends to prevent malicious actions from both the internet and the local network.
![[w2n4firewallSections.png]]
Here, it separates the internet from end user PCs, as well as both from existing servers. This means that an attack originating from any of the 3 regions are unable to affect anything outside of its own region.

A firewall applies a set of **policies**, which either allow or deny traffic. This generally consists of a blocklist - which allows everything that isn't explicitly blocked - or an allowlist - which denies everything that isn't explicitly allowed.

An example firewall policy could look something like this:

| Rule | Type | Source Address | Destination Address | Destination Port | Action |
|:----:|:----:| -------------- | ------------------- | ---------------- | ------ |
|  1   | TCP  | *              | 192.168.1.*         | 22               | Permit |
|  2   | UDP  | *              | 192.168.1.*         | 69               | Permit |
|  3   | TCP  | 192.168.1.*    | *                   | 80               | Permit |
|  4   | TCP  | *              | 192.168.1.18        | 80               | Permit |
|  5   | UDP  | *              | 192.186.1.*         | *                | Deny       |

# Firewall types
## Packet filters (stateless)
- If a packet matches the packet filter's set of rules, the packet filter will drop or accept it.
- It doesn't consider previous packets to pass through it, e.g. it can't have a rule to block an IP that attempts too many connections in a certain timeframe
- Stateless firewall policies often have to be very restrictive in order to prevent most attacks
## Stateful filters
- Maintains a table of all connections passing through it and can determine if a packet is either the start of a new connection, a part of an existing connection, or invalid.
- The session tables include information on each active connection, including IP addresses, ports, and packet sequence numbers
- Stateful firewalls are capable of allowing inbound TCP packets only when they are in response to a connection initiated within the internal network
## Application layer
- It can "understand" certain applications and protocols: it may inspect the contents of traffic, and block what it views as inappropriate content (e.g. websites, viruses, vulnerabilities, ...)
- It can do things like:
	- Block all web traffic containing certain words (censorship)
	- Remove all macros from Microsoft Word files attached to emails
	- Prevent anything that looks like a credit card number leaving a database
# Personal firewalls
A personal firewall runs on the workstation it protects, instead of a router/dedicated firewall device covering a network.
It provides basic protection, primarily for home/mobile devices, however can be disabled with any kind of rootkit.
# Pros and cons
Firewalls prevent straightforward attacks and information leakages, however they can be circumvented. Increasing their effectiveness substantially increases both the overhead (computationally/latency) and configuration time, and they may give a false sense of security. Essentially, a firewall is a necessary component in a network, but is not very effective on its own.