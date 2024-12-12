[[W8N1 - Memory hierarchy|Cache]] performance impacts [[W1N1 - Principles of computer architecture|processor performance]], specifically for load and store instructions.

Cache performance impacts processor performance following these equations:
$$
\begin{align}
\text{CPU time}&=IC\cdot CPI\cdot Clock\ period\\
\text{CPI}&=CPI_{ld/st}\cdot\frac{IC_{ld/st}}{IC}+CPI_{others}\cdot\frac{IC_{others}}{IC}\\
\text{CPI}_{ld/st}&\sim\text{Average memory access time}\\
\text{AMAT}&=\text{Hit time}+\text{Miss rate}\cdot\text{Miss penalty}
\end{align}
$$
This means there are 3 ways to improve memory hierarchy performance:
- Decrease hit time
- Decrease miss rate
- Decrease miss penalty
# Miss penalties
Miss penalties occur as on a cache miss, a [[W8N1 - Memory hierarchy#Cache write strategies|write-back]] cache must select and (potentially) copy back a victim block to the next level, and refill the required block from the next level. A write-through cache omits the copy back.
This delay can be calculated as:
- One or two block transfers of $B$ bytes.
- Sending a request from level $L_i$ to level $L_{i+1}$ takes $A$ cycles
- $L_{i+1}$ has a read latency of $R$ cycles
- Data transfer time per packet is $P$ cycles
- Packet size is $S$ bytes
- **Reading a block** then takes $A+R+P\cdot B/S$ cycles, as it takes $A+R$ to send the request and read the first packet, and send them in $B/S$ packets
- **Writing a block** takes $P+P\cdot B/S$ cycles, as it takes $P$ cycles to send the request and $B/S$ packets of write data
# Reducing cache miss rates
There are 3 types of cache misses:
- **Compulsory misses/cold misses**: when a block is accessed for the first time
- **Capacity misses**: when a block is not in the cache because it was evicted because the cache was full
- **Conflict misses**: when a block is not in the cache because it was evicted because the cache set was full
	- These only exist in [[W8N1 - Memory hierarchy#Direct mapped cache organisation|direct mapped]] and [[W8N1 - Memory hierarchy#Set associative cache organisation|set associative]] caches, in a fully associative cache all non-compulsory misses are capacity misses

Miss rates are very small in practice, often in the range of 0.5% for large caches, although this increases for smaller cache sizes. A rule of thumb is that miss rates change in proportion to the square root of cache size, e.g. $2\text{x cache}\rightarrow\sqrt{2}\text{ fewer misses}$. 
![[w9n1cacheMissRate.png]]
## Reducing compulsory miss rate
There are several approaches to reduce the number of compulsory misses:
1. **Large block size**: due to spatial locality, it is likely that other data in a block will be used soon, leading to each cold miss prefetching data. This comes at the cost of larger blocks reduces the number of blocks, so can increase the conflict and capacity miss rates. It also uses more memory bandwidth on each miss, which increases the miss penalty. This means there is a sweet spot for block size, which is often around 64 bytes.   ![[w9n1cacheMissesBlockSize.png]]
2. **Prefetching**: we can load data into the cache before it is used if it is likely to be used soon. This can reduce cold and capacity misses and doesn't typically increase the miss penalty, but uses more memory bandwidth and may increase conflict and capacity misses by displacing useful blocks (cache pollution). A prefetch buffer can be used to hold prefetched data which is moved to the main cache if it is used.
	- **Hardware prefetching** automatically prefetches cache blocks on a cache miss. This typically uses linear predictions, such as prefetching the next block from a memory access or looking for a pattern of block accesses (e.g. if 100, 200, 300 are accessed, then 400 can be prefetched)
	- **Software prefetching** uses compiler inserted instructions to trigger prefetches. This requires ISA support, and adds prefetch overhead instructions to compute the prefetch addresses and to perform the prefetch itself. This can prefetch data in loops or linked lists.
3. **High associativity caches**: if there are more options for block placement then there are fewer conflict misses. This can increase access (hit) time because the tag match takes longer, and can increase miss penalty as the replacement policy is more involved. Small caches are very sensitive to associativity, and more associativity always decreases miss rates, but there is very little difference between 4-way and fully associative cache miss rates. ![[w9n1missRateAssociativity.png]]

