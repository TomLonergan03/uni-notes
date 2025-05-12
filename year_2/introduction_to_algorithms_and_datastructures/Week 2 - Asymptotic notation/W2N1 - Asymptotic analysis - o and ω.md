**Asymptotic analysis** is a way of making quantitative statements about the efficiency of algorithms themselves (i.e. not language or platform dependent)

If we define functions
$T_I(n)$ = time taken by InsertSort on a list of length $n$
$T_M(n)$ = time taken by MergeSort on a list of length $n$
Then we can compare their performance
However, the values of these functions will differ depending on the list they are being applied to, e.g. InsertSort will perform $n$ comparisons on a sorted list while MergeSort will perform $n\textit{ log }n$ comparisons as it will still break each sorted array into its halves.

If we assume $T_I(n)$ and $T_M(n)$ are their respective functions worst case performance, then we could plot a graph:
![[w2n1PerformanceGraph.png]]

There are several ways to try to capture this behaviour:
1. $\forall n, T_M(n)<T_I(n)$
   This is untrue as as for small $n$,  $T_M(n)>T_I(n)$ 
2. $\exists N. \forall n \geq N. T_M(n)<T_I(n)$
   True, but doesn't capture the most important point:
3. Any implementation of MergeSort will eventually beat any implementation of InsertSort, therefore:
   $\forall c > 0. \exists N. \forall n \geq N. T_M(n)<cT_I(n)$


## o(g)
Method 3 states that $T_M$ is $o(T_I)$
This can be read as '$T_M$ is asymptotically smaller than $T_I$

In general, we say $f$ is $o(g)$ if
$\forall c > 0. \exists N. \forall n \geq N. f(n)<cg(n)$
Basically, for all 'advantages' $c$ given to the larger function the smaller function will still eventually beat it.
This is the equivalent of saying:
$\lim_{n\to\infty}\frac{g(n)}{f(n)} = \infty$

### Example 1
Is it true that $n^2$ is $o(n^3)$? 

Informally:
$\frac{n^3}{n^2} = n$ which trivially tends to $\infty$ as $n\to\infty$ 

Rigorous proof:
We must show that the $o$ formula is satisfied:
$\forall c > 0. \exists N. \forall n \geq N. f(n)<cg(n)$

If $c > 0$ then we have to pick an $N$ such that the inequality holds.
If $N>1/c$ then $\forall n \geq N$ we have
$cn^3 = cn \cdot n^2 \geq cN \cdot n^2 > c(1/c)n^2 = n^2$

### Example 2
Is it true that $100\sqrt n$ is $o(n)$?

Informally:
$\frac{n}{100\sqrt n} = \frac{\sqrt n}{100}$ which tends to $\infty$ as $n \to \infty$

Rigorously:
If $N>10000/c^2$ then $\forall n \geq N$ we have
$cn = c\sqrt n \sqrt n \geq c \sqrt N \sqrt n > c(100/c)\sqrt n = 100\sqrt n$

**How is $10000/c^2$ chosen?**
$n/(100\sqrt n) > 1/c$
$\sqrt n / 100 > 1 /c$
$n/10000 > 1 / c^2$
$n > 10000/c^2$
and since $n \geq N$ then
$N > 10000/c^2$

## ω(g)
$f$ is $o(g)$ means $f$ is asymptotically bigger than $g$
likewise, if we can say $g$ is $ω(f)$, or $g$ is asymptotically smaller than $f$

The formal definition is $f$ is $ω(g)$ if:
$\forall C > 0. \exists N. \forall n \geq N. f(n) > Cg(n)$