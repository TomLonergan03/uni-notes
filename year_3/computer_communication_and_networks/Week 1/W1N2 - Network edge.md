# Network edge
A network edge device is a host - either a client or a server (or both). Clients are generally devices such as PCs or mobile phones. Servers respond to requests from clients, and are often in datacenters, but can be small scale local systems also.

An end system connects to an edge router, with the capabilities of that router being decided on installation, based on factors such as required bandwidth, whether access must be shared between users or if specific hosts require dedicated connections.
# Access networks
## Digital subscriber line (DSL)
DSL uses existing telephone lines to transmit data at high frequencies, and voice at low frequencies. These two frequency ranges are separated using a DSL access multiplexer (DSLAM), with the voice frequencies going to the telephone network while the data goes to the internet.
## Cable based-access
Cable based access uses frequency division multiplexing (FDM) to allow many channels to transmit simultaneously by assigning each a frequency band. It can use TV lines, or dedicated internet lines.
# Wireless access networks
Shared wireless access networks connect end systems to routers via an access point or base station. A **wireless local area network (WLAN)** typically connect in or around a building (~100ft range) with an up to 10 Gbps transmission rate. A **wide area cellular access network** is provided by a mobile service provider, with ranges in the 10s of kilometres and lower transmission rates (~100 Mbps for 4G, 1 Gbps for 5G). An enterprise network consists of a mix of wired and wireless access points.
# Sending data
When an application wants to send a message, the network stack splits that message into smaller chunks (**packets**) of length $L$ bits. Then, these packets are transmitted into the access network at transmission rate $R$.
$$
\text{packet transmission delay}=\frac{L\text{ bits}}{R\text{ bits/s}}
$$
# Physical media
- A **bit** is a 0 or 1/logical low or high signal that propagates between transmitter/receiver pairs.
- In **guided media** signals propagate along defined channels, e.g. copper, coax, fibre
- **Unguided media** signals propagate freely, e.g. radio
## Twisted pair
A pair of insulated copper wires are twisted together, which reduces electromagnetic interference. Cat5 and Cat6 cables use a number of twisted pairs, reaching 100 Mbps-1 Gbps and 10 Gbps speeds respectively.
## Coaxial cable
Coax cables consist of an central copper wire, a layer of insulation, another layer of copper, and an outer sheath. This allows data to be bidirectionally transmitted, one way on each layer. Broadband cables are coax, with multiple frequency channels on each cable and can reach a speed of 100 Mbps per channel.
## Fibre optic cable
A fibre cable contains a glass fibre carrying light pulses, with each pulse being a bit. They are extremely high bandwidth, with transmission rates up in the 10-100+ Gbps range, have a low error rate as they are immune to EM interference, but are expensive and inflexible.
## Wireless radio
Instead of using a cable, radio/microwaves are used to carry signals, which propagate across the environment. This removes the need to physically connect devices to the network, but comes at the cost of variable signal strengths, interference, and obstruction by non-radio-transparent objects. Types include: terrestrial microwave (up to 45 Mbps), wireless LAN (up to 1 Gbps), wide area (up to ~1 Gbps), and satellite (up to 45 Mbps, has either high latency for geosynchronous orbits or higher cost/complexity for low-earth-orbit satellites).