![[w2n4bufferManager.png]]
The **buffer manager** determines when to transfer data between disk and memory, so that higher levels can assume that whatever data they are operating on are in memory.
# Buffer manager state
The buffer manager maintains two key datastructures:
- The buffer pool (MBs - GBs in size)
- Metadata: a hash table mapping page IDs to page metadata, most importantly the frame ID, whether the page is dirty and needs written back to disk on eviction, and the number of pins on that page
# Buffer pool
The **buffer pool** is an in-memory cache of disk pages, partitioned into frames. Each frame can hold a frame, and higher level code can request (**pin**) a page and release (**unpin**) a page. 

On a page request, if the page is not in the buffer pool the buffer manager will read it into memory, then return a pointer to the page. If the page is in the pool, the pointer can be returned immediately.

Higher levels need to explicitly release pages when they are done with them. A page may be used by multiple users simultaneously, and a page can only be removed from the pool when nobody is using it, but a page doesn't have to be removed as soon as nobody is using it.
## Pin implementation
```
function pin(pageno)
	if buffer pool already contains pageno then
		f = find frame containing pageno
		f.pinCount = f.pinCount + 1
		return address of frame f
	else
		f = select a free frame if buffer is not full or 
			a victim frame using the replacement policy
		if f.isDirty then
			write frame f to disk
		read page pageno from disk into frame f
		f.pinCount = 1
		f.isDirty = false
		return address of frame f
```
## Unpin implementation
```
function unpin(pageno, dirty)
	f = find frame containing pageno
	f.pinCount = f.pinCount - 1
	f.isDirty = f.isDirty || dirty
```
We don't need to write back the page on unpin as it may be pinned and modified again before it gets evicted, which would result in wasted disk operations.

If we have concurrent operations on the same page which perform conflicting writes, the concurrency control module with resolve them before the page is unpinned, allowing the buffer manager to assume everything is in order when it gets an `unpin(p, true)` call.
# Buffer replacement policies
When the buffer pool is full, a page is chosen for replacement by a **replacement policy**. The choice of policy can have a big impact on performance, as it affects the number of IO operations.
## Least recently used (LRU)
Under **least recently used (LRU)**, we keep track of when each page is used, and evict the unpinned page which was used the longest time ago. It is good when there are repeated accesses for popular pages, but can be costly as it needs to find the minimum on the last used attribute (even if using a priority queue for $O(\log n)$) as page accesses are frequent.
## CLOCK
**CLOCK** approximates LRU, and uses a circular buffer with a clock hand which points to the next page to consider for eviction.
```
while victim is not found:
	if frames[hand].pinCount == 0 then
		if frames[hand].referenced == 1 then
			frames[hand].referenced = 0
		else
			victim = address of frames[hand]
	hand = (hand + 1) mod N
```
![[w2n4clock.png]]
If we want to read page 10, first the hand skips page 3 as its pinned, then sets page 2 ref to 0 and moves on, then replaces page 8 as it's not been referenced. Page 10 is pinned, and it's ref is set to 1, and the hand advances to page 4.
### Sequential flooding
LRU and CLOCK are both susceptible to **sequential flooding**, where scans will wipe the entire buffer pool by inserting the entire scan at the head of the buffer.
## Most recently used (MRU)
**Most recently used (MRU)** replaces the most recently used frame, which is much more resilient to sequential flooding but fails to capture frequently used pages, instead just holding the first pages that were read.
# In practice
The DBMS knows the context of each page during query execution, so can provide hints to the buffer manager on whether a page is likely to be useful soon (**fix** it, e.g. nested-join loops) or say it is likely not to be used soon (**hate** it, e.g. pages in a sequential scan). Buffer pools may be partitioned into separate pools for tables, indexes, logs, etc. and pages may be prefetched by reading pages sequentially after the requested page.
# The OS
DBMSes don't use the filesystem to manage buffers and pages as the DBMS requires the ability to force flushing pages to disk in the correct order (which is required for recovery), and the DBMS has more information about query plans and access patterns of operators, which affects both page replacement and prefetching. In addition, different filesystems behave differently, so relying on it would reduce the portability of the DBMS.