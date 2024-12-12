# Andrew file system
**Andrew file system (AFS)** sacrifices the statelessness of [[W9N1 - Distributed storage#Network file system|NFS]] to improve scalability. AFS caches entire files, with `open()` fetching the entire file from the server, and `close()` flushing the file to the server if any changes occur. This means the server only has to do work for opening and closing, and reads and writes are faster as they are completely local. Files are updated on the server atomically, with the most recent write winning if concurrent writes occur. AFS keeps caches fresh using a stateful solution, where the server keeps track of which clients have a file open at any given time and sends a callback to each client if one of their open files change. If a client crashes, it evicts all data from the cache when it restarts. If a server crashes, it tells all clients to recheck all data before next open.
# Google file system
**Google file system (GFS)** is a scalable distributed file system for large distributed data-intensive applications at Google. It is implemented as a user-space library, with the goals being:
- Store large numbers of large files (millions of giga/terabyte files)
- Scalable access from hundreds of thousands of machines
- Fault tolerance as hardware failures are common at datacenter scales
- Location and fault transparency
- Files are normally appended with large amounts of data
- Random writes within files are very rare
- Files are mostly read, often sequentially
- Support concurrent appends
## Architecture
![[w10n1gfsArchitecture.png]]
### GFS coordinator
The **GFS coordinator** maps file names to chunks, and controls the chunkservers. Specifically, it manages:
- mapping chunks to chunk servers
- garbage collection of orphaned chunks
- migrating chunks between chunkservers

It also handles fault tolerance, with heartbeats to all chunkservers, and replicates the operation log across multiple servers.
### GFS chunkservers
The **GFS chunkservers** store the actual data using a Linux file system, and the client directly requests the data from them. A **chunk** is 64 MB of data, and is stored on local disks as normal files, along with a 32-bit checksum to detect corruption. A chunk is replicated 3 times on multiple chunkservers. A **chunk handle** is a globally unique 64 bit number that identifies a chunk, which is assigned by the coordinator on chunk creation and used by read and write requests.
### GFS client
**The GFS client** controls requests to the coordinator, and makes data requests directly to chunkservers. It caches metadata, but not data as it is expected not to need to read and write the same data often.
## Writes
A write occurs when the client requests a write from the coordinator, then sends the data to all 3 relevant chunkservers. The coordinator then grants a lease to one of the 3 chunkservers, which then tells the secondaries what order to apply writes in using sequence numbers, to ensure all replicas apply writes in the same order even if there are concurrent client writes. Once all replicas have applied the writes, the primary sends an acknowledgement to the client confirming the write is complete.
## Atomic record append
An atomic append only includes data, which is then appended to the file at least once atomically, at an offset of GFS's choosing. The selected offset is then returned to the client. When the primary receives the request, it checks if it fits in the current last chunk. If it does, it appends the data, tells the secondaries to do the same at the same byte offset, and replies with success to the client. If it doesn't, it fills the current chunk with padding, instructs the secondaries to do the same, and tells the client to retry the operation on the next chunk. If an append fails on any replica, the client retries the operation, so replicas of the same chunk may contain duplicates (hence the at least once guarantee).
## GFS metadata and operation log
Changes to the metadata on the coordinator are atomic, with the coordinator storing critical metadata changes in the operation log. The log defines a timeline that defines the order of concurrent operations, and is stored on the coordinator's local disk and replicated on remote machines. The coordinator recovers its file system state after a crash by replaying the operation log, and will only reply to a client after log entries are safe on its disk and replicas.
## GFS data consistency model
If a client writes to a part of a file that no other client is currently writing to, and the primary tells the client it was successful, all readers see the write. If multiple concurrent writes to the same chunk occur, and they all succeed, all readers will see the same content, but it may be a mix of the writes. If the primary doesn't tell the client the write succeeded, different readers may see different content, or none.

An atomic append may not be read in the same order, but it will always contain all writes if the append succeeded.

Applications are responsible for dealing with data inconsistency, e.g. using checksums and unique identifiers.
## Stale replica detection
If a replica is outdated, it is detected using a chunk version number. The coordinator increases the chunk version number and informs replicas when a new lease is granted on a chunk. When a chunkserver restarts after a crash, it reports its set of chunks and version numbers, which allows the coordinator to detect outdated chunks, which are then removed during regular garbage collection. The coordinator also provides the version number to clients, so they can check that they are accessing up to date data on reads.
## Handling faults
If a secondary fails, the primary may retry $n$ times, the client can choose to retry, and the coordinator may remove the secondary from chunk handle lists and replicate the data elsewhere.
If a primary fails, the coordinator will grant the lease to another secondary, and may remove the old primary from chunkhandle lists.
If the coordinator fails, on recovery it will replay the operation log to rebuild its state. It asks chunkservers what they store, and will wait for one lease time before granting any secondary a lease. Once this is done it can resume operations.