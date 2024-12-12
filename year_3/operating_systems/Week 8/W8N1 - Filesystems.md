# Primary roles of the OS
The OS maps [[W7N1 - Secondary storage|secondary storage blocks]] to files, in order to:
- hide hardware specifics, so the user can access different types of backing storage in the same manner
- allocate disk blocks as files are created or modified
- access data
- share data between users
- manage permissions on files
- maintain metadata about files
- [[W7N1 - Secondary storage#Block scheduling|optimise performance and/or flexibility]]

This is done using the **filesystem**, which maintains files - data - and directories - collections of files and other directories.
# Files
A file is an abstraction for disk resources, similar to [[W3N1 - Processes|processes]] for the CPU, or [[W6N2 - Virtual memory|virtual memory]] for physical memory.
A file consists of:
- Metadata, such as an owner, protection, size, last access time, etc.
- Data
There are many types of files, some specific to the file system, such as directories, symlinks, and devices, and some to be used by other parts of the OS, or in libraries or applications, such as programs or alphanumeric/binary data. File types are either encoded in the name or content, Windows uses file extensions to the name (`.exe`, `.txt`, etc.), while Linux deducts the file type from the content, using a "magic number" in the header.
## Basic operations
OSes provide an API for file operations:

| Unix                  | Windows                         |
| --------------------- | ------------------------------- |
| `create(name)`        | `CreateFile(name, CREATE)`      |
| `open(name, mode)`    | `CreateFile(name, OPEN)`        |
| `read(fd, buf, len)`  | `ReadFile(handle, ...)`         |
| `write(fd, buf, len)` | `WriteFile(handle, ...)`        |
| `sync(fd)`            | `FlushFileBuffers(handle, ...)` |
| `seek(fd, pos)`       | `SetFilePointer(handle, ...)`   |
| `close(fd)`           | `CloseHandle(handle, ...)`      |
| `remove(name)`        | `DeleteFile(name)`              |
| `rename(old, new)`    | `MoveFile(name)`                |
|                       | `CopyFile(name)`                |
## File access methods
File systems provide a number of different access methods:
- **sequential**: bytes are read/written one at a time, in order
- **direct**: given a byte number, that byte can be directly accessed regardless of where the previous operation was
- **record**: a file is an array of fixed or variable sized records (though this is usually implemented by applications instead)
- **indexed**: one file contains indexes that point to records in another file, e.g. databases. This is again usually implemented by applications

In Linux, only sequential and direct access are supported.
## File protection
File protection allows the file system to control who can access what, and how. This is often generalised, files become **objects** (the "what"), users become **principals** (the "who"), and read/write/etc. become **actions** (the "how").
There are two different protection models:
- **access control lists (ACLs)**: each object keeps a list of principals and their allowed actions
- **capabilities**: each principal keeps a list of objects and the principal's allowed actions
Linux primarily uses ACLs, and condenses them into three classes of principals: an owner, a group, and everybody else. This is the 3 sets of `rwx` bits for each file.
# Directories
Directories provide a way for users to organise their files, and a convenient way to namespace files for both the user and file system. Most file systems support multilevel directories (`/a/b/c`), and the notion of the current directory.

A directory is typically just a file that contains special metadata, and a **symbol table**: a list or hash table of `<file name, reference to file>`. The directory list is usually unordered, so tools like `ls` tend to sort the results before displaying them.
## Directory structure
Directories can be:
- **Single-level**: there is only one directory, so every file must have a unique name
  ![[w8n1singleLevelDirectory.png]]
- **Two-level**: there is one directory per user
  ![[w8n1twoLevelDirectory.png]]
- **Tree structure**: each user has a tree of directories and files. This leads to replicated files if multiple users need the same file
  ![[w8n1treeStructureDirectory.png]]
- **Acyclic graph structure**: Multiple directories can point to the same file, but cycles aren't allowed
  ![[w8n1acyclicStructureDirectory.png]]
## Path name translation
When a user wants to open the file, e.g. `/one/two/three`, the file system starts by opening the root directory `/` (which it always knows the location of), then searches for `one`, and once it finds it searches within `one` for `two`, and so on, with permissions checked at each step. The OS will cache commonly used prefixes to reduce the search times, as it takes a long time to make each disk access.
# File system data structures
At a high level, we must keep some information in storage, and some in memory.
- Storage
	- A **file control block (FCB)** for each file, containing details about the file as well as a unique identifier number for association with directory entries
	- A **boot control block**, containing information needed to boot an OS
	- A **volume control block**, containing volume details, such as number of blocks, size of blocks, a count for the number of free blocks, pointers to free blocks, a free FCB count, and FCB pointers
	- A **directory structure**
- Memory
	- A **mount table**, with information about each mounted volume
	- A **directory-structure cache**, which holds the directory information of recently accessed directories
	- The per-process **open file table**, which contains pointers to the appropriate entries in the system-wide open file table for the files that process has open
	- **Buffers**, holding file system blocks that are being read/written to a disk
# Inodes
An **inode** is an index node, which contains some metadata (file owner, access rights, etc.) and the locations of that file's blocks on disk.

A directory is a flat file of fixed size entries, with each entry containing a file name and an inode number. This is the contents of the directory's file data, *not* the directory's inode. Multiple directories can point to the same inode, or even the same directory can have different names pointing to the same inode.

There is a special inode for the root `/`, and one which contains all bad (broken/unreliable) blocks on that storage device.
## Data and metadata layout
An inode partition will consist of:
- a **superblock** specifying the boundaries of the next areas, and contains the head of the freelists of inodes and file blocks
- the **inode area**, which contains inodes for each file on the disk. All inodes are the same size, and the inode area is allocated when the disk is formatted, meaning there is a limit on the number of files which can be stored on a disk.
## Inode block lists
An inode contains a number of block pointers. Typically, there are 11-15 total pointers, with all but the last 3 pointing directly to 512kB blocks, and one single, double, and triple indirect pointers, which point to blocks of pointers, blocks of pointers to blocks of pointers, and so on.