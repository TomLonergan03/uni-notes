For a graph $G=(V,E)$:
- A subset $C$ of $V$ is *connected* (if $G$ is undirected) or *strongly connected* (if $G$ is directed)
- A *connected component* of $G$ is a maximum connected subset $C$ of $V$ where there is no other $C'$ of $V$ which strictly contains $C$
- $G$ is connected if it has only one connected component
- Each vertex of an undirected graph is in exactly 1 connected component
- For each vertex $v$ of an undirected graph, the connected component containing $v$ is the set of all vertices reachable from $v$

For an undirected graph $G$, both [[W10N1 - Depth-first search|DFS]] and [[W9N2 - Breadth-first search|BFS]] from a vertex $v$ will both exactly visit the vertices is the connected component containing $v$

# Implementation
```
connectedComponents(G):
	visited = [false]*|V|
	for all v in V:
		if visited[v] == false:
			print("New component")
			ccFromVertex(G, v)
```

```
ccFromVertex(G, v):
	visited[v] = true
	print(v)
	for all w adjacent to v:
		if visited[w] == false:
			ccFromVertex(G, w)
```