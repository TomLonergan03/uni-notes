**Seam carving** is an approach for resizing images by removing or duplicating **seams** (a connected sequence of pixels running from top-to-bottom or left-to-right across an image). This avoids some of the distortion caused by scaling, and avoids the loss of information from cropping.

For resizing an image, we want to find low energy seams, where there is little difference between seam pixels and their neighbours.

# Building a seam
When we construct a seam, we can select either the pixel directly below, one pixel left, or one pixel right of the current tip of the seam.
As we want to find low energy seams, we need an **energy function**, which calculates the energy of a pixel in some way.

The energy function is often a gradient score, such as using the Sobel operators:
$$
\begin{bmatrix}
	-1&0&+1\\
	-2&0&+2\\
	-1&0&+1
\end{bmatrix}
\text{ or }
\begin{bmatrix}
	-1&-2&-1\\
	0&0&0\\
	-1&-2&+1
\end{bmatrix}
$$
This is applied to each of the 3 colour channels then can be summed to get a total energy for each pixel.

# Computing an optimal seam
Assume we are looking for a vertical seam.
If an optimal vertical seam ends at a pixel $(m,j_m)$, then we know that $j_{m-1}$ was either $j_m$, $j_m-1$, or $j_m+1$.
Which of these it is depends on which of them is the endpoint of the optimal $(m-1) length seam.
Therefore we have 3 sub-problems of length $(m-1)$.

## Building the recurrence relationship
If we precompute $e_I(i,j)$ for every pixel $1\leq i\leq m$, $1\leq j\leq n$ then we have $O(m\cdot n)$ time for the precomputation and thereafter $O(1)$ to access each individual pixel.
We can then define $opt_I(i,j)$ to be the cost of the minimum-cost vertical seam from row 1 to pixel $(i,j)$.

We then produce the recurrence:
$$
\begin{align}
	opt_I(i,j)&=e_I(i,j)+0 \text{ if }i=1\\
	opt_I(i,j)&=e_I(i,j)+min(opt_I(i-1,j-1),opt_I(i-1,j)opt_I(i-1,j+1)) \text{ otherwise}
\end{align}
$$
# Implementation
We use a array of size $m\cdot n$ to store our $opt_I$ values in.
Each `opt[i,j]` will store the value of $opt_I(i,j)$ in it once it's been computed.
We also have a array `e` storing the $e_I(i,j)$ precomputed values.
As we want to know the actual path of the best seam, we have another array `p` where each element is either -1, 0, or 1; representing whether $j_i$ was $j_i$, $j_i-1$, or $j_i+1$ for the optimal seam to that pixel.

```
VerticalSeam(I,m,n):
	for j = 1 to n:
		for i = 1 to m:
			e[i, j] = energy(i, j)
		opt[1, j] = e[1, j]
		p[1, j] = 0
	for i = 1 to m:
		for j = 1 to n:
			op[i, j] = opt[i - 1, j]
			p[i, j] = 0
			if opt[i − 1, j − 1] < opt[i, j]:
				opt[i, j] = opt[i − 1, j − 1]
				p[i, j] = −1  
			if opt[i − 1, j + 1] < opt[i, j]:
				opt[i, j] = opt[i − 1, j + 1]
				p[i, j] = 1  
			opt[i, j] = opt[i, j] + e[i, j]
	max_seam = 2
	for j = 1 to n:
		if opt[m, j] < opt[m, max_seam]:
			max_seam = j
	print("Best vertical seam ends at cell (m, max_seam)")
```

The first nested loop runs in $O(m\cdot n)$ time, as does the main loop.
The last loop runs in $O(n)$ time.
This means that overall computing the best vertical seam occurs in $O(m\cdot n)$ time.
``