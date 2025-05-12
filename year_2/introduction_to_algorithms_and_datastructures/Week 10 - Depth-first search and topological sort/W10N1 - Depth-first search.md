Depth-first search is the opposite of [[W9N2 - Breadth-first search|BFS]], in that we first explore as far down a single chain of neighbours of the origin node, before expanding to the rest of the graph.

Animation of the order of which vertices are visited:
![[w10n1AnimatedDFS.gif]]

# Asymptotics
DFS performed on a graph $G=(V,E)$ takes $O(|V|+|E|)$ time.

# Implementation using a [[W4N1 - Classic datatypes#Stacks and queues|stack]]
```
dfs(G):
	visited = [false] * v
	S = new stack()
	for all v in V:
		if visited[v] = false:
			dfsFromVertex(G, v)
```

```
dfsFromVertex(G, v):
	visited[v] = true
	S.push(v)
	whie !S.isEmpty():
		u = S.pop()
		for all w adjacent to u:
			if visited[w] = false:
				visited[w] = true
				S.push(w)
```

`dfsFromVertex` can also be defined recursively, with no stack:
```
dfsFromVertex(G, v):
	visited[v] = true
	for all w adjacent to u:
		if visited[w] = false:
			dfsFromVertex(G, w)
```
This will however reverse the order that we visit the vertices adjacent to $v$.