Upper [[W2N4 - Protocol layering|layers]] would like to be able to use **reliable channels** - where data can be transferred in order, with no bits flipped or lost. A **reliable data transfer protocol** implements this using an unreliable channel along with software control to ensure it is reliable.
# Building a reliable data protocol
We will assume that we are only doing unidirectional data transfer (bidirectional is not much more complex though), using an unreliable lower layer channel that is modelled as a point to point immediate transfer. Our protocol will use `rdt_send()` to send a packet and `rdt_rcv()` to receive a packet, along with `udt_send()` to send an unreliable packet to the lower layer.
## Over a perfectly reliable channel `rdt1.0`
If the underlying channel is reliable then it is trivial to send data reliably, simply send it.
`rdt1.0`:
![[w5n1rdt1.png]]
There are 2 **finite state machines (FSMs)**, one for sending and one for receiving. The initial state is indicated by the dashed line, and solid lines represent a transition and associated action taken.

Here, the sender simply gets a packet when the next layer up calls `rdt_send()`, and it makes and sends that packet using `udt_send()`. The receiver gets the packet when `rdt_rcv()` is called by the lower layer, and it extracts the data and passes it up to the higher layer using `deliver_data()`
## Over a channel with bit errors
Here, all packets still arrive in order, but the data within them may change during communication.

To resolve this, the receiver can reply with an ACK (acknowledged) and NAK (not acknowledged) packet, depending on whether the the [[W4N3 - UDP#UDP segment structure|checksum]] is correct. If the sender receives a NAK then it will resend that packet again, and repeat until the receiver replies saying it is correct.
These are known as **stop-and-wait** protocols, as no new data is sent until the previous packet is delivered correctly.
`rdt2.0`:
![[w5n1rdt2.png]]

This protocol has a fatal flaw though, the ACK or NAK may also be corrupted. To solve this, the sender can simply resend the message if it receives a corrupted ACK or NAK, though this requires the packets to contain a sequence number (as now they can arrive out of order), and for the receiver to reorder packets once they arrive. In a stop-and-wait protocol, we can use either 0 or 1 for the sequence number, as we only ever have one packet in flight. 
`rdt2.1`:
![[w5n1rdt2.1sender.png]]
![[w5n1rdt2.1receiver.png]]

This can be simplified by removing NAKs, and instead each time a message arrives the receiver sends an ACK for the last correctly received packet. If the sender receives **duplicate ACKs**, then it knows that the most recent packet wasn't received properly, and can resend it.
`rdt2.2`:
![[w5n1rdt2.2sender.png]]
![[w5n1rdt2.2receiver.png]]
## Over a channel with bit errors and packet loss
If the channel is also capable of completely losing packets, then we must also detect that packet loss has occurred, and retransmit if that's the case. This is done by the sender having a timeout period that is long enough that it is likely (but not guaranteed) that the packet hasn't been delivered. If no ACK shows up in this time then the sender retransmits (this means that duplicate packets may be sent, but `rdt2.2` already can cope with that).
This gives us `rdt3.0`:
![[w5n1rdt3sender.png]]
(the receiver is the same as `rdt2.2`)

### The 4 cases of `rdt3`
![[w5n1rdt3Behaviour.png]]

# Pipelined protocols
`rdt3.0` is feature complete, but would have poor performance in a real world system, as only one packet is in flight at any one time. This might be fine for very short distances, but if a packet is travelling a long distance, such as from coast to coast in the US - a RTT of around 30ms - it would use a fraction of the available bandwidth. E.g. with that 30ms RTT, a 1 Gbps link, and 8 kb packets, it would be possible to send data at only 267 kbps, or 0.027% utilisation.

We can fix this by sending many messages simultaneously. This requires some changes to stop-and-wait protocols:
- The range of sequence numbers must be increased
- The sender will have to buffer unacknowledged sent packets
- The receiver may need to buffer received packets
## Go-back-N (GBN)
A **go-back-N (GBN) protocol** allows the sender to transmit up to $N$ packets without waiting for an ACK. $N$ is known as the **window size**, and GBN protocols are known as **sliding window protocols**.
![[w5n1gbnSequnceNumbers.png]]
The sender keeps the sequence number of the oldest unacknowledged packet, `base`, and the smallest unused sequence number, `sequencenum`. There are several intervals of sequence numbers:
- `[0..base]` have been sent and acknowledged
- `[base..nextseqnum]` have been sent but not yet acknowledged
- `[nextseqnum..base+N]` are available to send immediately on being passed data from an application
- `[base+N..]` are not available until an unacknowledged packet in the pipeline is acknowledged
![[w5n1gbnSender.png]]
![[w5n1gbnReceiver.png]]
When `rdt_send()` is called, the sender checks if it has an available sequence number within the window. If it does, it creates a packet with the data, sends it, and increments `nextseqnum`. When an ACK arrives, it is a **cumulative acknowledgement**, meaning that it says every packet up to and including the attached sequence number has been received correctly. For timeouts, there is a timer running from either the first packet resent following a failure, or from the last ACK received. If this timer times out, then the sender resends all packets that have been previously sent and aren't acknowledged. If there are no outstanding packets then the timer is stopped.

The receiver checks if each packet it received was correct, and if it is then it responds with an ACK for that packet's sequence number and passes the data to the upper layer. In all other cases (including if the sequence is out of order) the packet is discarded and an ACK for the last correctly received packet is resent.
![[w5n1gbnOperation.png]]
# Selective repeat (SR)
**Selective repeat (SR)** reduces the number of packets that the sender must retransmit, by having the receiver buffer out-of-order messages and request retransmissions only for corrupt or missing packets. Whenever a packet arrives, the receiver ACKs it (even if it's below the current window base)
![[w5n1srWindows.png]]
![[w5n1srOperation.png]]
The SR window must be less than or equal to half of the size of the sequence number space for an SR protocol, as if it is larger then there is no way for the receiver to know whether a packet is new data or a retransmission.

For a network which can reorder packets, a sequence number also can't be reused until it is assumed to no longer be in the network (for TCP this is around 3 minutes), as otherwise it would be possible for a packet to get delayed a minute and being retransmitted, the retransmitted packet getting ACKed, and the next packet going missing the receiver could then receive the old packet and ACK it even though it isn't the data the sender sent.