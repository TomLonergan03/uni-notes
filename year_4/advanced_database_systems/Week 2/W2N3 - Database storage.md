Most DBMSes store data as one or more files on a [[W2N2 - Disk usage|disk]].  Files consist of pages, which are the unit of transfer for disk reads and writes.

As reads and writes are expensive, they must be managed carefully, both to ensure pages used together are kept physically close together on the disk (for fast sequential reads), and to ensure pages are read in an order that minimises the number of CPU stalls while waiting for data to transfer.
# Disk space management
![[w2n3diskSpaceManagement.png]]
Disk space management maintains a mapping between page IDs and that page's physical location on disk, load pages into memory, and saving pages back to disk.

It exposes a simple interface to the buffer manager, with methods for allocating and deallocating pages, and reading and writing pages.
## Implementation
Most DBMSes use the local filesystem to contain the entire database in one large file. The DBMS relies on the OS and FS that sequential pages in this file are physically contiguous on disk, though a logical database file may span multiple FS files on multiple disks and/or machines. The disk space management keeps a mapping between page IDs to a filename and an offset within that file.