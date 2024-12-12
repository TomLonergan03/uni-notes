A [[W3N1 - Processes|process]] has an [[W5N1 - Physical memory#Memory abstraction|address space]]. When the process is loaded, that address space must be mapped to a physical address space. 
# Contiguous allocation
When a computer first boots, there is nothing in memory, so each process can simply take the next free region. As the computer runs though, processes terminate leaving holes in the physical memory for new processes to use. There are three ways to allocate a process to a hole:
1. **First-fit**: allocate the first hole that is big enough
2. **Best-fit**: search the whole list of holes and allocate the smallest hole that is big enough. This leaves the smallest leftover hole
3. **Worst-fit**: allocate the largest hole, again searching the entire list. This leaves the largest leftover hole
First-fit and best-fit have been found to be better than worst-fit in terms of speed and utilisation.
## Memory fragmentation
Programs can dynamically request or release memory. This means that a program may grow outwith its initially requested address space, or shrink leaving holes. This can be resolved by allocating more space than required, and by moving a program entirely when it uses up the full space, though this involves a lot of memory copies, which is not ideal.
There are two types of fragmentation:
1. **Internal fragmentation** is when a process has more physical memory than it is using
2. **External fragmentation** is when there is enough free memory to satisfy a request, but it is not contiguous. This can be fixed using compaction, which is relocating all programs to be closer together to free up space
# Swapping
It may be that there are more processes wishing to use memory than physical memory. In this case, programs can be swapped out (moved from memory to a disk) and swapped in again when needed
![[w5n2swapping.png]]