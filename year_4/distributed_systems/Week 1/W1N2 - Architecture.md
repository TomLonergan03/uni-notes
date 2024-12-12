# Layered
![[w1n2layered.png]]
Each layer performs an operation, and a layer will only interact with layers directly above or below it
# Three layer view
![[w1n2threeLayerView.png]]
- The **application interface layer** interfaces with users or external applications
- The **processing layer** contains the functions of an application
- The **data layer** stores the data that the client manipulates through the processing layer
# Service oriented
![[w1n2serviceOriented.png]]
Objects or microservices which are connected using **remote procedure calls (RPCs)**.
# Publish-subscriber
![[w1n2publishSubscribe.png]]
A node can subscribe to topics, and then whenever a node publishes an event to that topic, all subscribed nodes are notified.
# Centralised architecture
![[w1n2centralisedArchitecture.png]]
A centralised architecture follows a basic client-server model where a client requests a service from a server, which then responds with the result.
## Multitiered centralised architectures
![[w1n2multitieredCentralisedArchitecture.png]]
It is possible to divide parts of an application between a client and server in many ways, from *(a)* where the server provides everything to the client which only renders it, e.g. serverside rendered webpages, to *(c)* where the client performs some of the processing with the server doing other parts or validation of data, e.g. client rendered webpages, to *(e)* where data is stored locally and only backed up to a remote server, e.g. cloud storage backed desktop applications.
# Decentralised architecture
**Peer to peer (P2P)** systems have each node acting as both a client and a server. There are two types of P2P systems.
## Structured P2P systems
A **structured P2P system** adheres to a specific topology, e.g. a ring or tree. Here, the node that deals with a specific key/request will always be the same one.

![[w1n2chord.png]]
E.g. a **chord** is a distributed hash table where all data with a key $k$ will be stored at the smallest node with $id\geq k$.
## Unstructured P2P systems
An **unstructured P2P system** randomly selects the nodes that respond to a request, with specific data found using flooding or random walks. An example is [[W4N1 - Peer-to-peer file distribution#BitTorrent|BitTorrent]] for file sharing.
# Hybrid architectures
**Hybrid architectures** use a mix of centralised and decentralised architectures.
## Cloud computing
![[w1n1cloudComputing.png]]
**Cloud computing** uses a layered approach, with customers paying for what they use instead of paying for entire servers.
## Edge computing
**Edge computing** moves some of the processing from datacenters to servers or devices at the edge of the network, close to the point of access for the client. This can reduce latency, or be better for privacy as it keeps more data on device instead of being sent to the cloud.