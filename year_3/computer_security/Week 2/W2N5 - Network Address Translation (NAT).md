# Address space exhaustion
With IPv4 (addresses in the form `127.0.0.1`) there are $2^{32}$ IP addresses available. This means that there are not enough addresses for every network connected device on the planet.

The solution to this is **network address translation**:
- Each border router has an IP, and a mapping between requests and the actual internal IP address on that network.

![[w2n5NetworkAddressTranslation.png]]
Here, the two computers trying to connect both have the IP address `192.168.4.2`, but as the sender addresses the packet to the destination router there is no conflict, and a response is also possible as the outbound router replaces the from address as it routes the packet.