As we have seen, the transport layer provides process to process communication using the host to host communication provided by the network layer. We will now look at how the network layer achieves this.
# Overview
Unlike the above [[W2N4 - Protocol layering|layers]], the network layer is running on every host and every router between any two hosts on the same network. The network layer is divided into two components: the data plane and the control plane.

The primary role of the network layer is simple - it moves packets from a sending host to a receiving host. This requires two important functions to be performed:
1. **Forwarding**: a router must move each packet on its input links to the appropriate output link
	- This occurs very quickly, in the order of nanoseconds, so must be handled by hardware
2. **Routing**: the network layer must determine the route taken by packets as the travel from a sender to a receiver.
	- This occurs over seconds, so is usually implemented with software

Every router has a **routing table**, which maps fields in an incoming packet's header to the correct output link.
# The Internet's network layer
The internet provides only one service, known as the **best effort service**. This gives no guarantee that packets are received in the order they are sent, or even that they arrive at all.