UDP is pretty much as simple as a transport protocol can be. When it is given a message from the application layer it simply attaches a header containing source and  ports for [[W4N2 - Transport layer protocols#Transport layer multiplexing|multiplexing/demultiplexing]], adds 2 other small fields, does some light error checking, and hands it over to the network layer.
# Benefits of UDP
UDP gives more control to an application over what data is sent when, whereas TCP can delay packets for congestion control purposes, doesn't require a connection establishing handshake so can start transmitting data sooner, doesn't require any overhead at either end to maintain connection state, and has a small header overhead (8 bytes vs TCP's 20 bytes). This all makes it well suited for an application which wants to transfer large amounts of data, or to have very low latency transfers, and can handle losing some data or is willing to implement application level reliability guarantees (no small feat!). Common applications include streaming video/audio, internet based voice or video calls, network management, or [[W3N7 - DNS|DNS]].
# UDP segment structure
![[w4n3udpSegment.png]]
The header contains the port numbers, length of the segment including the header, and a checksum which is calculated by summing with overflow every 16 bit word in the segment, and storing the 1s complement of that number in the checksum. The receiver then can sum every word in the segment including the header, and this will equal `1111111111111111` if there have been no errors, and if any other sum is achieved then there was corruption of the data in a link or router somewhere along the journey.