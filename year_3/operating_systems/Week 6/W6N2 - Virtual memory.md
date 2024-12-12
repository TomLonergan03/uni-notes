# Paged virtual memory
When we have a larger logical address space than can fit in physical memory, we may have a program which requires more memory than exists on the system. We can use a [[W6N1 - Non-contiguous memory allocation#Paging|page system]] to be able to **swap** memory out to backing storage.
![[w6n2pageTableSwap.png]]
When a page is not in memory, its corresponding page table entry has the invalid bit set. When a process tries to access an invalid page $(1)$, it triggers a page fault exception $(2)$, at which point the OS checks an internal list of swapped pages. If the requested page is not there, then the access is invalid and the original software raises an exception. If it is swapped, then the OS will find an available frame in physical memory (or evict a current frame to disk), request the swapped page to that frame $(3,4)$, schedules another program, and once the page is swapped in it sets the valid bit $(5)$ on the page table and restarts the original program from the start of the instruction that caused the page fault $(6)$.
![[w6n2pageFaultHandling.png]]
# Demand paging
**Demand paging** only brings a page into memory when it is first accessed, meaning that on a program starting, only the first page of code is loaded from disk, while the rest waits until it is needed. **Anticipatory paging** attempts to predict and preload pages before they are needed.
Pages may be clustered, so loading one page causes other connected pages to also be loaded.
Demand paging can be expensive, and heavily depends on storage latency.
# Page replacement algorithms
When memory is full, we must choose a page to evict. We want to evict the page that will lead to the least future page faults. The best victim is a page that will never be accessed again, next best is one that will not be needed for a while. Unmodified pages are also good options, as they do not need to be written back to disk.
## First-in-first-out (FIFO)
This is similar to the [[W4N4 - Scheduling algorithms#First-come first-served (FCFS)|FCFS scheduling algorithm]]. 
In an example with 3 physical frames and 5 virtual frames, the following occurs

| Page requested | Head | Middle | Tail | Fault occurs? |
| :------------: | ---- | ------ | ---- | ------------- |
|       0        | 0    |        |      | Yes           |
|       1        | 1    | 0      |      | Yes           |
|       2        | 2    | 1      | 0    | Yes           |
|       3        | 3    | 2      | 1    | Yes           |
|       0        | 0    | 3      | 2    | Yes           |
|       1        | 1    | 0      | 3    | Yes           |
|       4        | 4    | 1      | 0    | Yes           |
|       0        | 4    | 1      | 0    |               |
|       1        | 4    | 1      | 0    |               |
|       2        | 2    | 4      | 1    | Yes           |
|       3        | 3    | 2      | 4    | Yes           |
|       4        | 3    | 2      | 4    |               |
Here, 9 faults occur.
### Belady's anomaly
We would expect that as we get more frames then less page faults occur. This is not necessarily true in practice.
# Least recently used (LRU)
Here, we replace the page that hasn't been used for the longest duration. Now, we still generate 9 page faults (in this case, though it is generally better in the long run):

| Page requested | Head | Middle | Tail | Fault occurs? |
| :------------: | ---- | ------ | ---- | ------------- |
|       0        | 0    |        |      | Yes           |
|       1        | 0    | 1      |      | Yes           |
|       2        | 0    | 1      | 2    | Yes           |
|       3        | 3    | 1      | 2    | Yes           |
|       0        | 3    | 0      | 2    | Yes           |
|       1        | 3    | 0      | 1    | Yes           |
|       4        | 4    | 0      | 1    | Yes           |
|       0        | 4    | 0      | 1    |               |
|       1        | 4    | 0      | 1    |               |
|       2        | 4    | 2      | 1    | Yes           |
|       3        | 4    | 2      | 3    | Yes           |
|       4        | 4    | 2      | 3    |               |
LRU is a good algorithm which is frequently used in the real world.
## Implementing LRU
The hardware page table is extended with page reference (if a page has been accessed) and page modified (if that access was a write) bits. We also keep a history/counter for each page in software.
# Second chance
Second chance is a FIFO variant, but now if a page is used it gets sent to the back of the queue.
We examine pages in FIFO order, starting from the beginning and consider a reference bit $R$. If $R=0$ then the page has not been referenced
1. If $R=0$, remove the page and go to 3.
2. if $R=1$, set $R=0$ and send to the back of the FIFO list, (giving it a second chance), and go to 1.
3. Add a new page at the end of the queue, with $R=1$
If a replacement hasn't occurred, then we revert to pure FIFO on the second pass.
(Second chance is usually implemented using a circular buffer)