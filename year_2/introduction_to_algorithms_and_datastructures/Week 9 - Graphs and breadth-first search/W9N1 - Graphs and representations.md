# Graphs
A graph is a mathematical structure consisting of a set of vertices ($V$) and edges ($E$).
A graph is undirected if for every edge $(v,w)$ there also exists an edge $(w,v)$. If this doesn't hold then it is undirected.
A directed graph is represented with arrows:
![[w9n1DirectedGraph.png]]
while an undirected graph is represented with lines:
![[w9n1undirectedGraph.png]]

## Uses
Graphs can represent a variety of (especially but not only) networks, such as road maps (with streets as edges and junctions as vertices), computer networks (computers are vertices and network connections are edges), or the internet (webpages are vertices and hyperlinks are edges)

# Graph representations
## Adjacency matrix
If a graph has $n$ vertices then it can be represented by a $n$ x $n$ 2D array, where a value at $(i,j)$ is 1 if there is an edge connecting vertex $i$ to vertex $j$ , otherwise it is 0.

## Adjacency list
A graph with $n$ vertices can also be represented by an array with length $n$, with each entry containing a list of all adjacent vertices.

## Comparison
If we have a graph with $n$ vertices and $m$ edges, and $out(v)$ is the number of edges beginning at $v$:

|                               | Adjacency matrix | Adjacency list   |
| ----------------------------- | ---------------- | ---------------- |
| Space                         | $\Theta(n^2)$    | $\Theta(n+m)$    |
| Check if $w$ adjacent to $v$  | $\Theta(1)$      | $\Theta(out(v))$ |
| Visit all $w$ adjacent to $v$ | $\Theta(n)$      | $\Theta(out(v))$ |
| Visit all edges               | $\Theta(n^2)$    | $\Theta(n+m)$                 |

Generally, an adjacency list is faster and smaller than a matrix, except if the main operation is checking adjacency of 2 vertices. Also, an adjacency list could still be faster if the graph is large and sparse.

# Sparse and dense graphs
Given a graph $G=(V,E)$ we know $m\leq n^2$.
$G$ is *dense* if $m$ is close to $n^2$.
$G$ is *sparse* if $m$ is much smaller than $n^2$.