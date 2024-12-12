The link layer controls individual transmissions between **nodes** - hosts, routers, or switches. A node sending data encapsulates a [[W7N1 - The network layer|network layer]] datagram in a **link layer frame**, and sends it to the next device in the [[W9N1 - Large scale routing|route]].
A link layer protocol may provide a number of services:
- **Framing**: nearly all link layer protocols encapsulate network-layer datagrams in frames before transmission.
- **Link access**: a **medium access control (MAC)** protocol specifies when a frame may be transmitted on the link. In a point-to-point link (e.g. an Ethernet cable directly connecting a computer to a switch) this is very simple, as transmissions may be made whenever the link is idle. They are more complex when many nodes share a single broadcast link, such as coax.
- **Reliable delivery**: similar to transport layer protocols such as [[W5N2 - TCP|TCP]], the link layer may provide guarantees that a link layer transmission makes it to the end of the link. This reduces the need for costly end-to-end transmissions, but has overhead, so links with low error rates (mainly wired) tend not to include it, while higher error wireless connections sometimes include delivery guarantees.
- **Error detection/correction**: while the Internet's network and transport layer protocols have simple error detection in their checksums, the link layer can use more sophisticated error detection such as detecting where in the frame the error has occurred, and possibly even correct errors instead of just forcing a retransmission.
# Network adaptors
Typically, the link layer is implemented in a **network adaptor**, or network **interface card (NIC)**. The heart of the network adaptor is the link layer controller, which is a single special purpose chip that implements the link layer protocol in hardware, along with some link layer software to interface with the card and to tell it what address to send the frame to.
# Error detection and correction
## Single parity bit
A single parity bit can be used to detect if an odd number of bit error occurs in some data. If $d$ bits are sent, then by adding a parity bit set so that there are an even number of $1$ bits in the $d+1$ total message means that if the receiver sums the number of $1$ bits it can check if errors occurred. This is fine if bit errors are rare and independent, but we need a more robust approach.
## Two dimensional parity
By dividing the $d$ bits of of data into $i$ rows and $j$ columns, and storing parity for each row and for each column, we can now detect any combination of two bit errors. We can also correct single bit errors, by looking for the intersection of the row and column without parity, and flipping that bit back to what it should have been.
![[w10n12dParity.png]]
## Checksums
By treating the $d$ bits of data sent as a sequence of $k$ bit integers, we can then simply sum them and store that result as the checksum. The [[W4N3 - UDP#UDP segment structure|internet checksum]] is based on this approach. Checksums require little overhead or processing power, so are used in software layers.
## Cyclic redundancy check
The **cyclic redundancy check (CRC)** is a more compute intensive error detection technique. CRC works by having the sender and receiver agree on an $r+1$ bit integer, known as a **generator**, then when the sender wants to send data $D$ of length $d$ bits it chooses $r$ additional bits $R$ and appends them to $d$ such that the resulting $d+r$ bit pattern is exactly divisible by $G$ using modulo 2 arithmetic, without carries in addition or borrows in subtraction. This means that addition and subtraction are identical, and both are equivalent to the bitwise XOR of the operands. $R$ can be calculated as:
$$
R=\text{remainder}\frac{D\cdot2^r}{G}
$$
A CRC check can detect all consecutive bit errors of $r$ or fewer bits, as well as any odd number of bit errors.