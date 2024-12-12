The internet is a public network, so routers see all traffic that passes through them. Along with this, packet headers must their contain source and destination in order to be routed. This allows routers, or people who have compromised them, to identify who is talking to who.
This is often not desirable.

There are 4 main methods:
1. Dining cryptographers
2. Crowds
3. Chaum's mix
4. Onion routing

# 3 party dining cryptographers (3DC)
The story goes:
Three NSA cryptographers are having dinner. At the end of the dinner they are informed that the dinner has already been paid for. Now, either the NSA paid for the dinner, or one of the cryptographers did. They want to know if it is the NSA that paid, or one of them, **without** revealing the identity of the paying cryptographer.

## 3DC Protocol
1. Each pair of cryptographers privately flip a coin, where heads = 1 and tails = 0
	- This means each cryptographer sees two results
2. Each participant announces the results as follows:
	- **If they did not pay:** the XOR of the two coin flips they observed
	- **If they did pay:** the negation of the XOR of the two coin flips they observed
3. If the XOR of the 3 announcements is 0 then the NSA paid, whereas if the result is 1 then one of them paid.
### nDC protocol
The 3DC protocol generalises to any group size $N>2$ (nDC):
If a sender wants to broadcast a message $M$. For each bit, $m_i$, of $m$:
1. every pair of users generate a random bit $(0,1)$ 
	- this means each user observes $n-1$ bits
2. each user (except the sender) announces the XOR of all $n-1$ observed bits
3. the sender announces the XOR of all $n-1$ bits and $m_i$
4. The XOR of all announcements = $m_i$

### Limitations
The DC protocol is impractical:
- It requires pairwise secret keys between participants to share random bits
- It requires large amounts of randomness
- Only one person can transmit at a time
- Any participant can launch a denial of service attack by either incorrectly performing the protocol, or by transmitting simultaneously with another participant

# Crowds
This is far less secure than nDC, but is also far more practical. (Similar to [[W3N3 - Symmetric encryption systems#One-Time Pad (OTP)|one time pads]] compared to [[W3N3 - Symmetric encryption systems#Advanced Encryption Standard (AES)|AES]], and a pattern that occurs throughout cryptography).

![[w6n2Crowd.png]]
A **crowd** works by having a group of $m$ users, with $c$ of those $m$ neighbours being corrupted.
An initiator that wants to request a webpage creates a path to the server by:
1. The initiator selects a forwarder from the crowd and sends him his request
2. The forwarder delivers the request directly to the server with probability $1-p_f$, otherwise it repeats the process, selecting a new forwarder, and so on
Each node records where it got a packet from, and where it sent it to. Therefore, when the server responds, it sends the reply to the last node in the crowd, which sends it to the node that it received the initial message from, and so on until it reaches the initiator.

A crowd is not resistant against an attacker that can see the whole network traffic.

# Chaum's mix
![[w6n2ChaumsMix.png]] 
Every user has the public key for the mix, as well as the public key of the server they want to communicate with.

They first encrypt their message with the server's public key, then attach the address of the server their trying to talk to. This is then encrypted with the mixes public key.

When the mix receives this message, it decrypts the outer layer and finds an encrypted message, and an address to send that message to. It then waits until it has enough messages pending, shuffles their order, and sends each out.

This means that:
- the order of messages arriving at the mix is not related to the order of messages leaving
- each message arriving at the mix is decrypted, meaning that the inbound and outbound messages should have no common data

Messages must also be padded to a consistent length.
Each user also generates dummy messages, to prevent an an attacker sending $n-1$ messages to a mix with capacity $n$, which would allow them to link the sender of the message to its recipient.

By providing a return address which is also a mix, the return address is also anonymised.

To guard against a mix being corrupted, a message is sent through a sequence of mixes. A single honest mix guarantees anonymity against an attacker controlling the whole network provided it applies:
- message padding
- buffering
- dummy messages

## Limitations
- Asymmetric encryption is inefficient
- Dummy messages are inefficient
- Buffering can introduce high delays, depending on how active the network is

## Mixing strategies
- **Timed mixing:** the mix receives packets for a fixed time period, then mixes and sends everything it has received
	- Number of messages sent is not fixed, time delay is fixed
	- This means you know any message that is sent will be delayed for no longer than that set amount of time
- **Threshold mixing:** the mix receives packets until it hits a fixed threshold, then sends them
	- Number of messages is fixed, time delay is not
	- You do not know how long your message may wait, but you do know that a specific amount of mixing has occurred
- **Pool mixing:** the mix has both a timer and a threshold. The mix waits until it hits that threshold, then it sends a set fraction of the pool every time the time elapses.
	- Guarantees there are always some messages in the queue, so there is always a threshold level of mixing, and generally there is not a long wait either.
- **Continuous mixing:** instead of the mix deciding the delay, the client does. If a delay is selected intelligently i.e. from a probability distribution such as an exponential distribution, and with a long enough mean so that enough other messages arrive at the mix between your message's arrival and transmission, then it is impossible to tell who's message is leaving the mix.

# Onion routing
Tor is a low latency, source routed (the sender determines the full path of the packet), which uses layered encryption and constant sized packets to hide the contents of messages. It defends against a local adversary, but not a global one (who can monitor the whole network).

![[w6n2OnionRouting.png]]

Onion routing works by:
1. The client opens a secure channel with an entry ("guard") node
2. The client asks the guard node to open a secure channel between the client and another node on the network
3. The client asks that node to open a secure channel between the client and an exit node
4. The client sends its traffic to the wider internet through that final secure channel

This means that the guard node knows who is connecting to it, the exit node knows where the traffic is going, but neither know both.

## Attacks
- **End to end correlation:** accessing any website will give a fairly unique series of packet timings based off of what needs to be downloaded. An adversary who controls both the entry and exit node is able to monitor this, and potentially determine what traffic is travelling between them
	- As Tor uses [[W3N3 - Symmetric encryption systems#Advanced Encryption Standard (AES)|AES in counter mode]], which is [[W3N3 - Symmetric encryption systems#^b682f9|malleable]], an attacker can XOR a bit or pattern into a packet at one end, and retrieve it at the other
- **Selective DOS:** if a malicious guard is used, and there exists a malicious exit node, the guard can kill the connection every time it is established unless the exit node is finding some correlation. The client will reconnect on a new route, and this can repeat thousands or millions of times until the client gets a connection with the compromised exit node
- **Website fingerprinting:** heuristics such as time between packets, number of packets send, and more can be used to create a semi-unique fingerprint for each website. This doesn't attack the Tor network at all, instead can just monitor local traffic before it reaches the guard node.
	- The modern state of the art deep neural networks are able to reach high 90s% accuracy, however this falls off as the number of sites needing to be identified increases
	- [Base rate fallacy](https://en.wikipedia.org/wiki/Base_rate_fallacy)
	- Generally, the classifier is trained on a limited selection of websites. On the internet, there are enough sites that it is fairly likely that there will be overlaps in site fingerprints
	- Defences tend to involve the server cooperating, which is often difficult to achieve
