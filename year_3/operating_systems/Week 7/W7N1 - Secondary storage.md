# Memory hierarchy
![[w7n1memoryHierarchy.png]]
While registers, cache, and main memory can be accessed in nanoseconds, a **solid state drive (SSD)** has an access time in the microseconds, and a **hard disk drive (HDD)** has an access time measured in milliseconds. This multiple order of magnitude increase requires a different approach to memory accesses, so secondary storage is accessed in **blocks** - multiple byte units, e.g. 512B or 4kB. The CPU also no longer directly accesses it, instead requesting specific regions from the appropriate [[W2N2 - Devices#Device drivers|device driver]].
Secondary storage is:
- **large**, 500-8000GB and greater
- **cheap**, often below £0.035/GB
- **slow**, ms or us
- **persistent**, so data is not lost when power is lost
- **rare to fail**, hard drives often have a **mean time between failure (MTBF)** of many years (if you have a lot of drives there are still regular failures, e.g. 100,000 drives with a MTBF of 3 years results in 1 failure every 15 minutes)
# Hard disk drives
![[w7n1hddPhysicalStructure.png]]
A HDD consists a number of spinning platters, divided into tracks, which are then divided into sectors. Sectors are then read and written using a head, of which there is one per platter. Historically, to read a sector from disk, the OS would request a certain head, track, and sector to be read, but this requires the OS to keep track of a lot to locate a sector. Instead, we can use a **logical block address (LBA)**, which is mapped onto all sectors of the disk sequentially.
![[w7n1logicalBlockAddressing.png]]
## Disk performance
Disk performance is dependent on:
- **Seek time**: how much time it takes to move the disk arm to the correct track.
	- This is not changing quickly, as it is limited by physics
- **Rotational latency**: how long it takes for a sector to rotate under the head
	- This is slowly increasing, but is also limited by physics
- **Transfer time**: how quickly data can be transferred from the surface of a platter to the disk controller
	- This is increasing relatively quickly

When the OS uses the disk, it tries to minimise these costs, using a number of approaches:
- Increasing file block size can reduce seeking
- Co-locating related items reduces seeking, this includes:
	- Blocks of the same file
	- Data and metadata for a file
- Keeping data or metadata in memory to reduce physical disk accesses, but this uses memory which may be better used elsewhere
- Prefetching blocks before they are requested hides slow disk accesses, but has a risk of loading unneeded files into memory
- Block scheduling
### Block scheduling
As applications request data accesses from the OS, the OS maintains request queues, from which it generates transfer commands to/from the disks. The OS can modify the order of those block requests being satisfied, to minimise wait times, or ensure fairness. There are many approaches, including [[W4N4 - Scheduling algorithms#First-come first-served (FCFS)|FCFS]], or prioritising the request with the shortest seek time from the current location (**shortest seek time first (SSTF)**), or by scanning from one edge of the platter to the other and repeating (**SCAN** - travel from left to right, then right to left, servicing requests in both directions - or **C-SCAN** - travel both directions, but only service requests when travelling rightward).
#### Selecting a scheduling algorithm
When there is only one pending request at a time, all algorithms behave like FCFS. SCAN and C-SCAN perform better for systems with heavy loads on disks, as they reduce the chance of a program starving due to a delayed disk access.
#### In Linux
In Linux, the block scheduling algorithm is a [[W2N1 - OS Structure#Kernel functionality extension|module]], allowing for easy customisation and replacement depending on use-case. The main block scheduler is **deadline**, which uses a variant of C-SCAN with two priority queues, one for high priority requests such as OS requests, and one for application requests. There are additional options, such as **NOOP**, which is optimised for SSDs, which do not have seek times in the same way as HDDs.
# Solid state drives
SSDs consist of an array of NAND flash, along with a number of flash controllers, for reading and writing them, as well as some processing.
![[w7n1ssdPhysicalStructure.png]]
An SSD reads a page at a time, and is typically 4kB. An SSD can handle upwards of 100,000 reads/s, and has 50-1000x lower latency than magnetic disks, along with 1-10x higher read throughput. In addition, read time is mostly independent of device geometry, although a flash controller can only read from one of its NAND flash cells at a time.
An SSD write also occurs at a page at a time, but flash media must be erased before it is written, and this occurs as a per block (64-256 pages) operation. It takes around 1ms to erase a block, and each block can only be erased a certain number of times before it fails, typically 10,000-1,000,000 times. This means that the SSD will buffer a number of writes, and will often write data back to a different block than it read it from to ensure equal wear across all cells using the **flash translation layer (FTL)**.
# SSD vs HDD
- **Capacity**: a flash SSD costs around 2x the cost per GB relative to an HDD
- **Energy**: SSDs are typically more energy efficient than HDDs, with SSDs taking 1-2 watts while HDD require ~10 watts
- **Physical resistance**: an SSD has no moving parts, while a HDD can have issues if it experiences vibrations or acceleration.
# Storage device management
Before a disk is first used, it must be initialised. This consists of formatting, creating the partition table (pointers and metadata about different partitions on that device), and creating boot code, which instructs the computer how to read from the disk as the OS boots.