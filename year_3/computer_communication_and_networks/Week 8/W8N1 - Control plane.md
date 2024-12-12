The network **control plane** is responsible for determining what should be in [[W7N2 - Routers|routers']] forwarding tables to ensure packets get to their destination host as quickly as possible. There are two approaches to this:
- **per router control**: a routing algorithm runs on every router, and so routers talk to each other to determine what the best routes are ^bf97d5
- **logically centralised control**: a single routing controller determines the best routes, and tells all routers on the network what their forwarding tables should be ^b1a5b9
# Routing algorithms
A **routing algorithm** determines good (according to some metric) paths from senders to receivers across a network of routers. We can represent a computer network as a graph ($G$), with nodes ($N$) being routers and edges ($E$) being the physical links connecting them.
![[w8n1networkGraph.png]]
Each edge has a "cost" to use it, which could be distance, link speed, or monetary cost. For any edge $(x,y)\in E$ we have: $c(x,y)=\text{cost of that edge}$ and $c(x,y)=\infty$ if there is no edge $(x,y)$. We will only look at undirected graphs (i.e. $(x,y)=(y,x)$ and $c(x,y)=c(y,x)$). We say that node $y$ is a neighbour of node $x$ if $(x,y)\in E$. A path through a graph is a sequence of nodes $(x_1,x_2,...,x_p)$ such that each pair $(x_1, x_2),(x_2,x_3),...,(x_{p-1},x_p)$ are in $E$. The cost of that path is the sum of all edge costs along that path, i.e. $c(x_1, x_2),c(x_2,x_3),...,c(x_{p-1},x_p)$. At least one of these paths is the **least-cost path**, and a good routing algorithm will find paths at least close to this one.
Routing algorithms can be divided in several ways:
- **static vs dynamic**: a static routing algorithm is computed once, and rarely changes, while a dynamic algorithm runs regularly and automatically
- **load sensitive vs insensitive**: a load sensitive algorithm dynamically varies link costs based on current congestion levels, while a load insensitive algorithm does not.
- **centralised vs decentralised**: very similar to [[W8N1 - Control plane#^bf97d5|per router]] and [[W8N1 - Control plane#^b1a5b9|logically centralised]] control
# The link-state routing algorithm
A **link-state algorithm** uses knowledge of every link cost in the network.This is done by having every router broadcast the current edge costs to every other router on the network. A standard link-state routing algorithm is [Dijkstra's algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm), which computes the least-cost path from a source node to every other node on the network. Dijkstra's algorithm is iterative, and after the $k$th iteration the least-cost path to $k$ nodes is known, and those $k$ nodes have the $k$ smallest path costs.
![[w8n1linkstateOscillation.png]]
When all routers dynamically update simultaneously, it can lead to oscillations like the above, where traffic is routed along more costly paths than optimal. This can be fixed by having each router randomise the time at which it sends out a link advertisement.
# The distance-vector routing algorithm
The **distance-vector (DV) algorithm** is iterative, asynchronous, and distributed. Each node receives some information from its neighbours, performs some calculations and broadcasts its results to its neighbours (*distributed*). Each node repeats this until no more computation needs done (*iterative*; interestingly the DV algorithm also self-terminates, no halt signal is sent, instead computation just stops once all the work is done). The algorithm is *asynchronous* in that nodes do not have to update together, instead each does work as and when it can.

The DV algorithm is based on the **Bellman-Ford equation**:
$$
d_x(y)=\text{min}_v\{c(x,v)+d_v(y)\}
$$
or "the shortest path from $x$ to $y$ consists of the shortest path from $x$ to some node $v$ plus the cost of $v$ to $y$".

The DV algorithm has each node $x$ maintain the following routing information:
- for each neighbour $v$, the cost $c(x,v)$
- node $x$'s distance vector $D_x=\left[D_x(y):y\in N\right]$, $x$'s estimated distance to every other node in the network
- the distance vector for each of its neighbours $v$, $D_v$

Whenever a node receives an updated distance vector from a neighbour, it uses the Bellman-Ford equation to update its own distance vector. If node $x$'s distance vector changes as a result of the update, it transmits its own new distance vector to all neighbours. The $v$ that is used in the Bellman-Ford equation is used as the next hop router for that destination.

![[w8n1countToInfinity.png]]
It is possible for loops to form if link costs increase, where one node $x$ routes through $y$ to get to $z$, while $y$ routes through $x$ to get to $z$. In the above case, when $c(x,y)$ goes up to 60, then $y$ recalculates its shortest path to 6, but this is based on $z$ saying it has a $c(z,x)=5$ path available. $z$ then gets this update, and recalculates $c(z,x)$ to 7, and so on until the costs have counted up to be more than 50 (44 iterations). This is known as the **count-to-infinity problem**. This can be partially fixed by having $z$ lie to $y$ and say it has no route to $x$ because its route to $x$ goes via $y$ (the **poisoned reverse**). This is not perfect though, as larger loops can still form.
# Comparison
DV and LS algorithms differ on a number of key metrics:
- **Message complexity**: LS requires each node to be sent the cost of every link in the network, or $O(|N|+|E|)$ messages, and when a link cost changes this must all be sent again. DV only sends messages between neighbours at each iteration, and these messages need only to be sent after a link cost changes, significantly reducing the number of messages needed
- **Speed of convergence**: LS takes $O(|N|^2)$ to calculate all routes, while DV converges slowly and can have routing loops until the network converges
- **Robustness**: single corrupt routers can influence DV more significantly, as every router depends on every other routers communication, causing incorrect values to diffuse through the whole network.