Breadth-first search visits all vertices reachable from a start vertex $v$ in the following order:
1. $v$
2. all neighbours of $v$
3. all neighbours of neighbours of $v$
4. and so on

I find BFS can be visualised as if the search is a flood starting at the start node and travelling out at the same speed in all directions.
Animation of the order of visiting vertices when performing BFS from $a$.
Grey vertices are in the queue, black vertices have been explored.
![[w9n2AnimatedBFS.gif]]

# Asymptotics
BFS performed on a graph $G=(V,E)$ takes $O(|V|+|E|)$ time.

# Implementation using a [[W4N1 - Classic datatypes#Stacks and queues|queue]]
```
bfs(G):
	visited = [false] * v
	Q = new queue()
	for all v in V:
		if visited[v] = false:
			bfsFromVertex(G, v)
```

```
bfsFromVertex(G,v):
	visited[v] = true
	Q.enqueue(v)
	while !Q.isEmpty():
		v = Q.dequeue()
		for all w adjacent to v:
			if visited[w] = false:
				visited[w] = true
				Q.enqueue(w)
```
