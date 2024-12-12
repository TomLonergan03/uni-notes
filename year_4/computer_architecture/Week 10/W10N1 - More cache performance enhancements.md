As we've seen, there are many ways to improve cache performance, such as by [[W9N1 - Cache performance enhancements#Reducing cache miss rates|reducing cache miss rates]]. The other terms of the [[W9N1 - Cache performance enhancements|cache performance equation]] can also be lowered.
# Reducing cache miss penalty
Cache performance can be improved by reducing the amount of time it takes when a cache miss occurs. There are several methods:
1. **Victim caches**: by using very small (often 8-32 entries), fully associative caches, evicted lines can be cached, which reduces the impact of conflicts in [[W8N1 - Memory hierarchy#Direct mapped cache organisation|direct mapped]] or [[W8N1 - Memory hierarchy#Set associative cache organisation|set associative]] caches (as usually there are only a few entries that end up conflicting). If an entry is found in the victim cache it is swapped back into the main cache. Victim caches are rarely used in modern processors. ![[w10n1victimCache.png]]
2. **Prioritise reads over writes**: the value of a load is likely to be used soon, while latency of writes do not matter much. Writes can be queued in a write buffer, and reads are prioritised with writes only occurring when the pipeline is idle or the write buffer is full. A read to an address with a pending write will be a hit in the buffer. ![[w10n1readOverWrite.png]]
3. **Early restart** and **critical word first**: when a read misses, the processor only needs the requested word or byte but must wait for the whole block to be brought into cache. Sending the word to the processor as soon as it arrives in cache speeds this up, and reordering the memory access to send the requested word first can speed it up further. These optimisations are always applied in modern processors.![[w10n1criticalWordFirst.png]]
4. **Non-blocking cache**: when a cache miss occurs, it may still be able to service other hits. This is known as **hit under miss**, and more generally can be **hit under $n$ misses** where $n$ is the number of pending misses allowed. This can only work when the cache and memory can service multiple requests concurrently, and is particularly valuable in [[W6N3 - Dynamic instruction scheduling|dynamically scheduled]] processors. **Miss status handler registers (MSHRs)** are a hardware structure that contain the address of the block being waited on, have the ability to merge multiple requests to the same block, and a destination register for the load. ![[w10n1nonblockingCaches.png]]
5. **Second layer (L2) caches**: as the gap between L1 and main memory speed increases, more caches can be added between L1 and memory. The L1 miss penalty becomes the L2 hit access time if it hits in L2, but the miss penalty is higher if L2 also misses. L2 cache has much higher capacity (256KB-4MB), an access time of 10-20 cycles, and a higher associativity (often 8-16 ways) and a higher miss rate than L1 as L1 will have already extracted most of the locality from the requests that pass through it. The performance of the memory system now takes the form:   $$
   \text{Average memory time}=\text{Hit time}_{L1}+\text{Miss rate}_{L1}\cdot(\text{Hit time}_{L2}+\text{Miss rate}_{L2}\cdot\text{Miss penalty}_{L2})$$The **local miss rate** is the number of misses divided by the number of requests for a specific cache, while the global miss rate is the number of misses divided by the number of requests from the CPU and represents the aggregate effectiveness of the cache hierarchy. ![[w10n1l2CacheSize.png]]Furthermore, L3 caches are common on laptop/desktop/server processors, and have a 30+ cycle access time, very large capacity (2-20+ MB), and very high associativity (16-32 ways).
# Decrease hit times
Decreasing the hit time will improve performance in the vast majority of cases, as most accesses are hits. There are several approaches:
1. **Small and simple caches**: small caches are compact, so have short wire spans and low latency. Simple caches with low associativity or direct mapping have few tags to compare against requested data. These optimisations reduce hit time at the cost of increasing miss rate. 
2. We also have to translate addresses from [[W6N2 - Virtual memory|virtual to physical]], with a **translation look-aside buffer (TLB)**. ![[w10n1tlb.png]]
	The TLB consists of:
	- **Valid bit**: whether the current entry exists
	- **Dirty bit**: whether the page has been modified
	- **Read/write/execute bits**: give permissions for the page and are checked on every access
	- **Tag**: the virtual page number
	
	A TLB miss or privilege violation causes an exception, which is handled by the operating system. There are often separate
	This leads us to our second optimisation, **virtual address caches**. Addresses must be translated at some point, but we can do this translation after the L1 cache. This reduces the hit time for L1 cache entries.![[w10n1virtualAddressCache.png]]
	There is a special case where processes may share memory by mapping the same physical memory into each of their address spaces. This means that if this is supported (all operating systems expect it to be), then the address must be translated before the L1 cache.
3. **Virtually indexed, physically tagged (VIPT) caches**: if we ensure the cache index is entirely within the set of untranslated page offset bits of an address, we don't need to translate the page before looking it up. ![[w10n1vipt.png]]
   As an L1 index and a TLB lookup take approximately the same time, doing them in parallel can halve the time for a cache hit. VIPT caches are very common in modern processors.
# Summary

|        Technique        | Miss rate | Miss penalty | Hit time  | Complexity |
| :---------------------: | --------- | ------------ | --------- | ---------- |
|    Large block size     | Reduced   | Increased    |           | Reduced    |
|   High associativity    | Reduced   |              | Increased | Increased  |
|      Victim cache       | Reduced   | Reduced      |           | Increased  |
|    Hardware prefetch    | Reduced   |              |           | Increased  |
|    Compiler prefetch    | Reduced   |              |           | Increased  |
| Compiler optimisations  | Reduced   |              |           | Increased  |
| Prioritisation of reads |           | Reduced      |           | Increased  |
|   Critical word first   |           | Reduced      |           | Increased  |
|   Non-blocking caches   |           | Reduced      |           | Increased  |
|        L2 caches        |           | Reduced      |           | Increased  |
| Small and simple cache  | Increased |              | Reduced   | Reduced    |
|     VIPT L1 caches      |           |              | Reduced   |            |
