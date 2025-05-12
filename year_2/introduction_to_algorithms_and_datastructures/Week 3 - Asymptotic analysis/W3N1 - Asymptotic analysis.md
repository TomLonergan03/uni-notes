There are many possible ways to compare algorithmic cost (for various kinds of cost). We must use a cost model - a definition of how we intend to measure cost. 
Different cost models are useful for different purposes, e.g. we may care about memory usage or runtime or number of disk operations and so on.

Even if we were to decide to focus on runtime, there are many different cost metrics. E.g. for sorting algorithms we could use the number of comparisons, or the number of actual instructions performed by the CPU, or the amount of time taken to execute each program. 