**Dijkstra's algorithm** is used to find all single source (starting from one node, ending at every other node) shortest paths for any graph or directed graph without negative weight edges.
![[w11n1DijkstraExample.gif]]
Red nodes have had all their neighbours visited.

The algorithm works by:
1. First marking the source node as having distance 0 and all other nodes having either undefined or infinite distance, before marking each neighbouring node with it's distance along the shared edge between them. 
2. It then selects the node with the shortest distance, and sets all of its neighbours' distances to the sum of its own distance plus the distance along their shared edge **only if the newly calculated distance is less than the node's current distance**.
3. Once this is completed, the node is marked as visited, and returns to step 2.
4. The algorithm terminates either when all nodes are visited, or in the case that we only want the distance to a specific node, when that specific node is selected in step 2.

If we want to recover the shortest path to each node, not just the distance, then for each node we must also store the previous node along the path to it. This is as if the shortest path to a node c is a->b->c, then the shortest path to b must be a->b, and therefore knowing c is preceded by b, and b by a, is enough to reconstruct the path to c.

# Implementation
Using a [[W6N1 - Heap datastructure|min heap]] for a priority queue to store all unvisited nodes using their current distance as key, with an implementation that allows us to reduce the value of a key, allows us to get the closest unvisited node in $O(\text{lg }n)$ time.

```
InitialiseSingleSource(Graph, source):
	for each v in Graph.vertices:
		distance[v] = infinity
		previous_node[v] = None

Relax(Graph, (u, v)):
	if distance[v] == infinity:
		distance[v] = distance[u] + Graph.weight[u, v]
		previous_node[v] = u
		to_be_visited.insertItem(distance[v], v)
	else if distance[v] > distance[u] + Graph.weight[u, v]:
		distance[v] = distance[u] + Graph.weight[u, v]
		previous_node[v] = u
		to_be_visited.reduceKey(distance[v], v)

Dijkstra(Graph, source):
	InitialiseSingleSource(Graph, source)
	to_be_visited.insertItem(0, source)
	distance[source] = 0
	while !to_be_visited.isEmpty():
		(current_distance, u) = to_be_visited.extractMin()
		for v in out(u):
			Relax(Graph, (u, v))
```

# Asymptotics
`InitialiseSingleSource` takes $O(n)$ time.
Initialising the queue with the source is $O(1)$ time.
A vertex can only be added to the heap once and removed from the heap once, so $O(n\cdot \text{lg }n)$ covers all `to_be_visited.insertItem` and `to_be_visited.extractMin` calls.
Having covered `insertItem` calls, a call to `Relax` takes $O(1)+T_{reduceKey}(n)=O(1)+O(\text{lg }n)=O(\text{lg }n)$ time.
`Relax` can be called at most twice for each edge (once from either end), so all `Relax` calls happen in $O(m+m\cdot\text{lg }n)$ time.

Therefore, Dijkstra's algorithm on a graph with $n$ vertices and $m$ edges **runs in $O((n+m)\cdot\text{lg }n)$ time overall**