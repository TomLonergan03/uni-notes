# DFS Forests
A [[W10N1 - Depth-first search|DFS]] traversal of a graph will produce, if the vertices and edges visited are recorded, a forest (number of trees) whose edges contain all edges traversed during the traversal and whose vertices are all vertices of the graph.

For example, a graph and its forest:
![[w10n2dfsForest.png]]

# Topological sorting
Topological sorting allows us to order a graph into trees so that each vertex comes before all vertices it points to. This is the same as the forest produced by a DFS, however a topological sort can only be performed on a Directed Acyclic Graph (DAG), a graph which has no cycles (a sequence of edges that returns to the starting vertex).

This is useful, for example, where we have a list of tasks where some depend on others having been completed:
- Task 0 must be completed before Task 1 can be started.  
- Task 1 and Task 2 must be done before Task 3 can start.  
- Task 4 must be done before Task 0 or Task 2 can start.  
- Task 5 must be done before Task 0 or Task 4 can start.  
- Task 6 must be done before Task 4, 5 or 7 can start.  
- Task 7 must be done before Task 0 or Task 9 can start.  
- Task 8 must be done before Task 7 or Task 9 can start.  
- Task 9 must be done before Task 2 or Task 3 can start.

We can represent these dependencies as a graph:
![[w10n2taskGraph.png]]
and then find a topological ordering, for example:
8 -> 6 -> 7 -> 9 -> 5 -> 4 -> 2 -> 0 -> 1 -> 3
There are often many topological orderings for a given graph.

# Asymptotics
Topological sort performed on a DAG $G=(V,E)$ takes $O(|V|+|E|)$ time.

# Implementation
```
topologicalSort(G):
	state = [unvisited]*|V|
	L = new LinkedList()
	for each v in V:
		if state[v] = unvisited:
			sortFromVertex(G, v)
	print(L)
```

```
sortFromVertex(G, v):
	state[v] = visited
	for all w adjacent to v:
		if state[w] = unvisited:
			sortFromVertex(G, w)
		else if state[w] = visited:
			print("G has a cycle")
			exit()
	state[v] = finished
	L.insertFirst(v)
```