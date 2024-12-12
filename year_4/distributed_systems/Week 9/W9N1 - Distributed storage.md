# File system basics
A file is an array of persistent bytes that can be read or written. There are two interpretations of [[W8N1 - Filesystems|file systems]]:
1. A file system is a collection of files, this is also known as a file system image.
2. A file system is the part of the OS that manages those files. There are many local file systems, e.g. ext2, ext3, ext4, ntfs, xfs, zfs. Files are a common abstraction across all file systems.

Files need names to be accessed, and there are three types of names:
- Unique **inode** numbers identify one specific file within a file system image. 
	- Different file systems may use the same number, and numbers can be recycled after deletes.
	- Inodes are stored in a known, fixed block location on disk in an **inode file**. Each inode contains the address of a file, along with its size.
- A path is how to find a specific file starting from a root directory.
- A file descriptor identifies a specific file that a program has opened.
# File system on disk structures
Alongside the data block containing the actual data for a file, there are a number of metadata structures stored on disk.
## Inode table and inode block
An inode is typically 256 bytes (or 128 bytes on a few file systems). An inode block is a 4 KB disk block that exclusively stores inodes, so each inode block stores 16 inodes. The inode blocks combine to form the inode table.
![[w9n1inodes.png]]
## Inode, data, and indirect blocks
An inode contains the metadata for a file, such as type, owner, permissions, size, time of access and creation, and a multilevel index pointing to the file contents. This consists of **direct pointers** which point to blocks containing data, and **indirect pointers** which point to blocks containing more pointers to data. If we have 12 direct pointers, 1 single indirect pointer, and 1 double indirect pointer in an inode, we have a maximum file size of $(12+1024+1024^2)\cdot4KB=4GB$. This gives fast performance on small files, with 2 disk reads necessary: one to read the inode and one to read the data. Large files will have worse performance as the tree of indirect pointers will each need read from disk.
## Directories
![[w9n1directory.png]]
**Directories** are used to organise files, and are usually stored as regular files with the entry inodes in data blocks, with a bit in the inode distinguishing directories from files.
## Data bitmap and inode bitmap files
![[w9n1dataInodeBitmap.png]]
The **data and inode bitmap files** identify free and used blocks/inodes. They are usually set to 1 if allocated and 0 if free.
## Superblock
![[w9n1superblock.png]]
The **superblock** is generally the first block on the disk, and contains information about the file system such as number of blocks and inodes, block size, and more.
# Flows
## Creating a file
If we create an empty file at `/foo/bar`, we:
1. Read the root inode to get its block address
2. Read the root block to get the inode of `foo`
3. Read the `foo` inode to get its block address
4. Read the `foo` block to check `bar` doesn't already exist
5. Read the inode bitmap to select a free inode for `bar`
6. Write that bit in the inode bitmap to 1
7. Write `bar`'s inode to foo
8. Read `bar`'s inode
9. Write `bar`'s inode to initialise it with no data blocks (as the file is empty)
10. Write `foo`'s inode to update metadata, such as number of files contained
## Opening a file
If we open the previously created `/foo/bar`:
1. Read the root inode to get its block address
2. Read the root block to get the inode of `foo`
3. Read the `foo` inode to get its block address
4. Read the `foo` block to get the inode of `bar`
5. Read the `bar` inode and create a file descriptor
## Reading a file
If we read from the opened `/foo/bar` file descriptor:
1. Read the `bar` inode to get the block address of the requested data
2. Read the correct `bar` data block to get the data
## Writing a file
If we append to the opened `/foo/bar` file descriptor:
1. Read the `bar` inode
2. Read the data bitmap file to select a free data block
6. Write that bit in the inode bitmap to 1
3. Write the data to that new data block
4. Write the `bar` inode to update it to also point to the new data block
## Closing a file
If we close the opened `/foo/bar` file descriptor:
1. The file descriptor is marked as invalid, no disk accesses have to occur
# Types of file systems
**Local file systems** allow processes on the same machine to access shared files on that machine.
**Network file systems** allow processes on different machines to access shared files on a single server.
**Distributed file systems** should provide:
- Fast and simple crash recovery, where both clients and servers may crash
- Transparent access which uses normal UNIX semantics
- Reasonable performance which can be scaled with the number of clients
# Client-server communication
There are several strategies for client-server communication:
1. Wrap regular UNIX syscalls using RPC:
	- `open()` on the client calls `open()` on the server, which returns `fd` to the client
	- `read(fd)` on the client calls `read(fd)` on the server, which returns the data back to the client
	- If the server crashes and reboots, then the fds returned by the server will not be valid. This can be resolved by running some complex crash recovery protocol when the server reboots or by persisting fds on the server disk, but these are both complicated and can cause issues if a client crashes or misbehaves
2. Create a stateless protocol with each request containing all information required for a desired operation. This works across crash and reboot with no correctness issues, at the cost of slower performance as a file must be opened on the server every time a client wishes to read or write it ^446b5e
# Network file system
**Network file system (NFS)** is a protocol for networked file systems. We will look at NFSv2, as NFSv4 is much more complex.
## NFS architecture
![[w9n1nfsArchitecture.png]] 
![[w9n1nfsDiagram.png]]
NFS governs the communication between clients and servers through a virtual file system, but does not manage the layout of data on the server's disks, instead leaving that to any UNIX compliant file system.

NFS roughly uses [[W9N1 - Distributed storage#^446b5e|approach 2]], with the client using the standard UNIX API which then call RPC-based APIs on the server. Clients maintain their own file descriptors, with a client calling `open()` creating a local fd. The local fd contains a file handle of `<volume ID, inode number, generation number>` (with generation number being incremented each time an inode number is reused), and the current offset of the fd. When the client reads or writes, it sends the file handle, offset, and size to the server.
## NFS caching
With NFS, data can be cached in 3 places: server memory, client memory, and client disk. All of these caches must be kept in sync.
There are a number of problems that NFS caches must overcome:
- An NFS server often buffers writes to improve performance, which means the server may acknowledge writes before it is actually written to disk. If the server crashes, these writes can be lost, so either the performance hit from not using the optimisation must be accepted, or more expensive persistent memory may be used for caching.
- If a client has cached data and modifies it, the server must be made aware by the client flushing cache entries to the server, and should choose the correct time to flush it. NFS does this when a file is closed, and optionally in other cases such as when it is low on memory. NFS does not ensure the file flushes are atomic, as it sends one block at a time, so if two clients flush at the same time it can mix data.
- Clients must ensure that their cache is up to date with the server. This must be done statelessly as NFS is stateless, so clients check if their cached copy is current before using data. The recheck only involves exchanging timestamps of files, so it is much faster than fetching the data. This is done by sending a file `stat` request to the server, which returns the last timestamp.
	- It was found that `stat` requests account for 90% of server requests, so the client caches `stat` results for 3 seconds, at the cost of clients being able to read data that is up to 3 seconds old.