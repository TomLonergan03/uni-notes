# Memory basics
There are some basic pairs of memory types:
- Sequential vs random access
	- **Random access memory (RAM)** can have any address accessed in constant time
	- **Sequential memory** must be read in order, and takes time to seek to a specific address, e.g. HDD and magnetic tape storage
- Volatile vs non-volatile
	- **Volatile memory** loses its contents when it loses power
	- **Non-volatile memory** preserves its contents between shutdowns, e.g. ROM, SSD, HDD
- Static vs dynamic
	- **Static memory** will keep its contents for as long as its powered, at the cost of taking up 6x more die area than dynamic memory
	- **Dynamic memory** loses its contents over time, usually around 1-10ms, and must be refreshed
# RAM structure
DRAM contains multiple banks. A **bank** consists of a 2D array of bits.
![[w10n2ramBank.png]]
A memory access involves selecting a bank, then a row, then a column. The area used by each decoder is $O(n\log n)$, where $n$ is the number of outputs. This means that for very small memories the area of the decoders dominate, while larger memories are dominated by the physical memory. This is why we tend to model small memories like register files as individual flip flops which are muxed, and large memories as RAMs.
# DRAM structure
![[w10n2dramStructure.png]]
Each DRAM bit is stored in a tiny capacitor, which leaks over time which is why refreshes are required.
## DRAM access
![[w10n2dramAccess.png]]
A DRAM chip is accessed by pulling the **row access strobe (RAS)** line low, then sending the row address, then pulling the **column access strobe (CAS)** line low and sending the column address. The chip then responds with the data, and there is a period where the RAS and CAS cannot be pulled low again as the refresh happens.

Reads destroy the bits that are read, so after each read the values are then written back to the row. A precharge period is needed before each memory access to allow the sense-amplifiers to read the value in the cells.
## Refresh operations
Rows are refreshed sequentially, using a refresh counter that selects the next row to be refreshed (essentially a cyclic counter up to $2^k-1$), and a refresh controller, which generates the appropriate RAS and CAS signals. A refresh is done by simply reading the data, which result in the data being written back to the cell afterwards.
## Optimising DRAM
DRAM access exhibits [[W1N1 - Principles of computer architecture#^00d1cb|spatial locality]], so neighbouring columns are likely to be accessed next. There are a couple of ways to speed up DRAM accesses using this fact:
- **Fast page mode DRAM**: we can allow multiple CAS cycles within each RAS cycle
- **Row buffers**: on read, we can copy the entire row into a buffer, which can then be accessed in multiple locations, freeing memory to start a new cycle

# Non-volatile memory (NVRAM)
NAND flash memory devices can be used for persistent file systems, or as a persistent last-level cache. This comes at the cost of a limited number of lifetime writes, a high cost per bit, and a high power consumption.