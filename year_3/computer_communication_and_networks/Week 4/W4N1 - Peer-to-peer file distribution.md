All the applications we've looked at so far - [[W3N5 - The Web and HTTP#The Web|the Web]], [[W3N6 - E-mail|email]], and [[W3N7 - DNS|DNS]] - have involved significant always on servers. We will now look at an example of a [[W3N1 - Network applications#Peer-to-peer (P2P) architecture|P2P]] service - distributing large files from a single server to a large number of hosts (called **peers**). In a traditional single server architecture, the server must send the entire file to each of the peers in turn, which uses a lot of server bandwidth, as well as resulting in a single point of failure in the network. By using a P2P architecture, once the server has sent part of a file to a peer, that peer can then also send it to another peer, and so on. 
As of 2019, the most popular P2P file distribution protocol is BitTorrent.
# Scalability
If we have $N$ peers wishing to download a $F$ bit file, with the $i$th peer having an upload rate of $u_i$ and a download rate of $d_i$, along with a server with an upload rate of $u_s$, we can make a simple model of file distribution performance. The **distribution time** is the time it takes for all $N$ peers to receive a copy of the file. We also assume that there are no bandwidth limits within the [[W2N1 - Network core|network core]], no latency, and that all clients and the server are devoting their full bandwidth to the file share.
## Client-server distribution time
For a traditional client-server architecture we can say the following:
- The server must transmit $F$ bits to each of the $N$ hosts, which takes $\frac{NF}{u_s}$
- Each client must download the full $F$ bits, so if $d_\min=\min\{d_1,d_2,...,d_N\}$ then the client that takes the longest to download a file will take $\frac{F}{d_\min}$ 
Combining this, we can say that the lower bound on the transfer $D_{cs}$ is
$$
D_{CS}\geq\max\left\{\frac{NF}{u_s},\frac{F}{d_\min}\right\}
$$
In fact, it is possible to schedule the downloads so that the lower bound is always achieved, by sending the file first to the client with the slowest download speed, then the next slower, and so on, so in this case we can say:
$$
D_{CS}=\max\left\{\frac{NF}{u_s},\frac{F}{d_\min}\right\}
$$
(n.b. this requires the server to have perfect knowledge over each client's download rate)

We can then see that if $N$ gets large enough the download time will always be governed by the number of clients (as a link with a download speed of e.g. 10 bits/s will certainly time out well before it ever attempts to download a file), so $D_{CS}\propto N$ - a thousandfold increase in the number of peers will lead to a thousandfold increase in distribution time.
## P2P distribution time
Reusing the same setup as before, but this time with a peer to peer server, we notice the following:
- The server must send every bit to at least 1 peer (as a peer can then redistribute that data), which takes $\frac{F}{u_s}$
- The peer with the lowest download rate will still take $\frac{F}{d_\min}$ to receive the full file
- The total upload capacity of the system as a whole is equal to the upload rate of the server plus the upload rates of each individual peer ($u_{total}=u_s+u_1+...+u_N$), so the system must upload $F$ bits to each of the $N$ peers, which cannot be done faster than $\frac{NF}{u_s+\sum^{N}_{i=1}u_i}$.
We can then say the lower bound on distribution time for P2P, $D_{P2P}$ is:
$$
D_{P2P}\geq\max\left\{\frac{F}{u_s},\frac{F}{d_\min},\frac{NF}{u_s+\sum^{N}_{i=1}u_i}\right\}
$$
Again, we have a strategy to distribute a file at this lower bound, by the server sending 1 bit to each client, so:
$$
D_{P2P}=\max\left\{\frac{F}{u_s},\frac{F}{d_\min},\frac{NF}{u_s+\sum^{N}_{i=1}u_i}\right\}
$$

Again, in practice this isn't feasible as chunks of files are redistributed instead of single bits, but the lower bound is still a good approximation.
## Comparing distribution time
Using the 2 above models, we can compare distribution times - assuming all peers have the same upload rate $u$, $\frac{F}{u}=1\text{ hour}$, $u_s=10u$, and $d_\min\geq u_s$.
![[w4n1p2pVsClientServer.png]]
We can see that the client-server minimum distribution time increases linearly and without bound, while the P2P minimum distribution time is not only always less than the CS system, but in fact is always less than 1 hour for any number of peers $N$. This shows that P2P systems are self scaling, as each peer adds capacity to the network as well as increasing demand.
# BitTorrent
BitTorrent is a popular P2P file distribution protocol. The collection of all peers participating in a particular file distribution is called a **torrent**. Peers in a torrent download equal sized **chunks** of the file from each other, with each chunk typically being 256 KB. Once a peer has downloaded the entire file it may leave the torrent (selfishly, known as **leaching**), or stick around and continue to upload chunks (altruistically, known as **seeding**). In addition, a peer may leave the torrent at any time with only some of the chunks downloaded, and may rejoin later.

Each torrent has a **tracker**, which keeps a list of all active peers. When a new peer (we'll call Alice) joins a torrent, it contacts the tracker, which selects a random subset of peers and sends their IPs to Alice who attempts to open concurrent TCP connections with all the IPs it received. Each successful connection is counted as a "neighbouring peer", and over time some of those peers will leave while others join and establish connections, leading to Alice's neighbouring peers fluctuating over time. Every neighbour will have a different subset of chunks of the file, so Alice will periodically request a list of the chunks each of its neighbours has. Alice then determines which chunks are held by fewest of her neighbours and requests them first (**rarest-first**), this means that each chunk of a file should end up roughly equally distributed.
Alice will also receive requests for chunks for each of her neighbours, and she selects the four that are sending her the highest rate of bits, and reciprocates by sending them their requested chunks - these neighbours are known as **unchoked**. She recalculates the top four every 10 seconds. Every 30 seconds, she also selects one random neighbour (in this case Bob) and sends it chunks - this neighbour is **optimistically unchoked**. If Alice makes it into Bob's top four uploaders and he will reciprocate, and may also make it into Alice's top four. This means that peers will gradually match up with other peers that are capable of uploading at comparable rates, and that a new peer will get chunks that it can trade with its neighbours. All other neighbouring peers receive nothing from Alice, and are **choked**.