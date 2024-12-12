The simple way to allocate memory is to give each program one [[W5N2 - Basic memory allocation#Contiguous allocation|contiguous]] region of memory. This will however, leave some gaps in memory that is unused but cannot be filled as it is too small for any program to fit there. We can instead use **non-contiguous memory allocation**: logical memory stays as a contiguous address space, but the physical memory backing it may be in several chunks. There are two main approaches to this: segmentation and paging.
# Segmentation
An address space is partitioned into variable size chunks called **segments**. These segments are based on logical units: stack, heap, data, etc. Each segment has a segment number, and a logical address is `<segment #, offset>`.
![[w6n1segments.png]]This requires hardware support from a **segment table**, which stores base/limit pairs. A logical address then consists of a segment number, which is the index for the table, and an offset, which is the offset from the segment base address.
![[w6n1segmentationTable.png]]
# Paging
Paging instead divides both the logical and physical address spaces into fixed, equally sized chunks.
![[w6n1pages.png]]This means that there are never any unfillable holes, as every hole can accept every page.
We call logical chunks **pages** and physical chunks **frames**. A logical address is divided into a page number, which indexes a page table to find the frame's base address, and a page offset which gives the final physical memory address.
![[w6n1pagingHardware.png]]
Now all that needs to be done is mapping $p$ to $f$, and now no limit check has to occur. Each process has its own page table, the operating system loads the correct page table each time a process is loaded to give the correct translations for that process' virtual address space.
![[w6n1pagingMultipleProcesses.png]]
## Paging advantages
- No external fragmentation can occur
- Internal fragmentation is dependent on page size, with worst case fragmentation occurring if a program needs n pages plus 1 byte, resulting in n+1 pages being allocated.
	- Most modern computers use 4kB, and many support 2MB and even 1GB
# Page table implementation
The page table is stored in memory, with the **page-table base register (PTBR)** in the CPU pointing to the page table address and the **page-table length register (PTLR)** in the CPU storing the size of the page table. This means that a data/code access requires two memory accesses, one to fetch the page table entry, and another to fetch the actual memory content. We can reduce the number of memory access by using **translation look-aside buffers (TLBs)**. These are fast-lookup hardware caches, that have entries in the form `<page #, frame #>`. If a translation exists in the TLB, then it can be used at no cost, but if it doesn't then the translation must be fetched from memory. The cache maintains translations for subsequent memory accesses, so replacement policies must be considered, and some entries may be wired down for permanent fast access.
![[w6n1tlb.png]]
## Effective access time (EAT) with a TLB
If we have a hit ratio of $\alpha$, we then have a miss ratio of $1-\alpha$. If we consider $\alpha=0.8$, and a memory access takes $100\text{ns}$, then we have:
$$
EAT=0.8\cdot100+0.2\cdot200=120\text{ns}
$$
If $\alpha=0.99$, and memory access still takes $100\text{ns}$, then;
$$
EAT=0.99\cdot100+0.01\cdot200=101\text{ns}
$$

## Memory protection
A **page table entry (PTE)** also has more information, such as protection bits, and valid bits. These bits state which types of access are allowed: read-only, read-write, or execute-only. If a program attempts to violate this, then a hardware trap to OS occurs. The valid bit states whether or not the PTE is a valid translation, allowing some entries to be discarded, e.g. if memory isn't full and there are some pages that haven't been used.
## Shared pages
We can [[W3N2 - Interprocess communication#Shared memory|share memory between processes]] by mapping pages in each of their address spaces to the same frame. This means that if multiple processes use the same library code it only needs to be loaded into memory once.
## Page table design
The page table can be very big. If we have a 32-bit logical address space and a page size of 4kB. In this case the page tables would have ~1 million entries ($2^{32}/2^{12}$). As each entry is 4 bytes, 4 MB of memory space is required only for the page table, which needs to be allocated contiguously in physical memory. This costs a lot, e.g. for 4GB of memory ~0.1% is used as a page table for each process.
### Hierarchical page tables
A **hierarchical page table** breaks up the logical address space into multiple page tables, then constructs another table that refers to each of those page tables.
![[w6n1hierarchicalPageTable.png]]
This means that if our application uses e.g. only the first block of pages, then only a small section of the page table needs allocated. Now, logical addressing consists of an outer page #, inner page #, and page offset.
![[w6n1hierarchicalPageAddressing.png]]
Modern systems tend to use four or five layer page tables (Linux uses 5).
