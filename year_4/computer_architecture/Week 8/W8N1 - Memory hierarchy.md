Memory speed has increased vastly slower year on year than processor speed has (between 1980 and 2010 it was 1.2-1.5x per year for processors, but only 1.07x for memory, resulting in a 10000x increase in clock speeds, but only 10x for memory speed).

This discrepancy in speed means that reducing latency of memory accesses can be very beneficial for processor performance. The primary way this is done is using a **memory hierarchy**, with smaller amounts of expensive but fast memory close to the processor, and larger amounts of cheap and slow memory further from the processor.

Memory hierarchies are effective due to **temporal locality** (a recently accessed memory location is likely to be accessed again in the near future) and **spatial locality** (memory locations close to a recently accessed location are likely to be accessed in the near future).

![[w8n1memoryHierarchy.png]]
# Cache block placement
A cache must have a way to map the location of a block from anywhere in memory to a location in the cache. 
![[w8n1cacheBlockPlacement.png]]
There are 3 methods:
- **Fully associative cache**: a block from memory can go anywhere in the cache. This allows for full usage of a cache, but makes finding a cached entry harder as all entries must be queried in some form.
  ![[w8n1fullAssociative.png]]
- **Direct mapped cache**: each block in memory can only go in one location in cache, e.g. block 12 can only be stored in block 4 ($12\mod{8}$) in the cache. This makes it simple to query the cache, but may result in some blocks going unused or being unnecessarily evicted.
  ![[w8n1directMapped.png]]
- **Set associative cache**: each block is mapped to one set, which consists of multiple blocks any of which can take the cached block. E.g. block 12 must be stored in set 0 ($12\mod{4}$), but can go in either block 0 or 1. This allows for more efficient use of the cache than direct mapping, while also reducing the number of look ups required compared to full associativity.
  ![[w8n1setAssociative.png]]
# Cache block identification
![[w8n1blockIdentification.png]]
Every block is identified by a tag, which is part of the memory address. The tag is stored alongside the block data in the cache, and are compared with the tag of a requested block (often in parallel across all blocks in a set). The block tag usually comprises the high-order bits of the memory address. The block offset is the location within the block of the byte being requested, and the index is the set where the block is found.
# Direct mapped cache organisation
![[w8n1directMappedOrganisation.png]]
Here, the index directly connects to the cache entry, so on an access the only thing that needs checked is the valid bit.
# Set associative cache organisation
![[w8n12waySetAssociativeOrganisation.png]]Here, the index is used to query both blocks in the set for the value, and which is used depends on which block has the matching tag.
# Cache block replacement
When a new block is loaded into the cache another block must be evicted. In a direct mapped cache, this is easy as there is only one choice of block to evict. In an associative cache, there are several approaches:
- **Ideal**: select the block that will not be used for the longest amount of time, but this requires knowledge of the future so is unrealistic
- **Random**: select a random block
- **Least recently used (LRU)**: select the block that has not been used for the longest time. This works well (due to temporal locality), but requires 1 bit of LRU state per set for 2-way associativity and more complex state machines for higher associativity
- **Not recently used (NRU)**: select a random block other than the most recently used. This performs better than random, and is much simpler than LRU though it performs worse.
## Pseudo-LRU
![[w8n1pseudoLru.png]]
**Pseudo-LRU** uses a binary tree per set that points to an approximation of the most recently used. Each time an access is made, the path through the tree to that access has its bits changed to point the other way from the actual access. It doesn't always evict the least recently used entry, but will get one of the older ones.
# Cache write strategies
When a write occurs, cache and memory must be updated. There are 2 approaches:
- **Write through**: all levels of cache and memory are written at the same time. This is rarely used.
	- This slows writes to the speed of the lowest level of the hierarchy
	- It generates more traffic to low levels, which tend to have lower bandwidth
	- Low levels are kept coherent with the higher levels, which is good for multi-processors
- **Write back**: only write back to the next level on block eviction. This requires a **dirty bit** for each entry, which is set when the block has been modified and will have to be written back.
	- Writes perform at the speed of the highest level
	- It generates less traffic
	- The lower levels can have stale data for some time

If a block is not found in the cache, then there are 2 approaches:
- **Write allocate**: bring the block into the cache then write to it. This can be good, but can also lead to filling caches e.g. if we are zeroing a large region of memory. This is usually used with write-back
- **Write no-allocate**: do not bring the block into cache, instead modify the data in the lower level. This is usually used with write through
# Multi-level caches
A lower level cache may keep a copy of blocks that are brought into higher level caches:
- **Inclusive caches**: have a copy of all data in higher-level caches. This wastes capacity of lower level caches, but simplifies other entities (such as processors) finding cache blocks.
- **Exclusive caches**: a block may reside in only one level of the cache hierarchy. This maximises the aggregate capacity of the hierarchy, but requires a uniform block size for all cache levels.