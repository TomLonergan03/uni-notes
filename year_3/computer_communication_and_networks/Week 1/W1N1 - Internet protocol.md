The **core** of a network contains the routers, switches, etc. that move packets around, while the **edge** are the devices which connect to that network and run network applications.

The internet contains:
- **hosts:** devices which are connected to the internet and runs network applications
- **packet switches** which forward packets to hosts or other switches/routers (a router uses a CPU, while a switch uses an ASIC. Routing is based on IP address, while switching is based on MAC address)
- **communication links** are the connections with the network. These can include fibre or copper lines, radio, and satellite communications. Different links have different **bandwidths** (quantity of data transferable per second).
- **networks:** collection of devices, routers, and links, which are managed by an organisation

The internet is a "network of networks", consisting of many interconnected ISPs which connect many edge networks, such as individual homes, businesses, and organisations.
# Protocols
A **protocol** governs the specific actions in response to specific messages/requests.

Protocols control the sending and receiving of messages, e.g. HTTP/HTTPS, TCP, UDP, IP, WiFi, etc. Protocols are layered up, e.g. TCP runs on top of IP, and ensures that messages sent are delivered, so you can simply send a packet using TCP without worrying about whether it goes missing in transit.
