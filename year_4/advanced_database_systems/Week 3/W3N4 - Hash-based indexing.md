Hash-based indexes are suitable for equality-based predicates (`SELECT * FROM Customer WHERE A = constant`) but do not support range queries. Some query operators generate many equality tests, e.g. `JOIN`. In general, [[W3N3 - Tree-based indexing|tree-based indexes]] are preferred since they cover more general range queries, while hash-based indexes are used for joins.

Static and dynamic hashing techniques exist, with similar trade-offs to [[W3N3 - Tree-based indexing#ISAM|ISAM]] vs [[W3N3 - Tree-based indexing#B+ tree|B+ trees]].
# Hash functions
A hash function needs to be lightweight (so non-cryptographic) and have a low collision rate to be useful for a DBMS. A simple hash function is $h(k)=k\mod{N}$, which guarantees the range of $h(k)$ to be $[0,...,N-1]$, and if $N=2^d$ we can effectively just use the least $d$ bits of $k$ only, though prime numbers work best for $N$.
# Static chained hashing
The hash index is a collection of $N$ successive pages, known as primary buckets, and each bucket has a pointer to a chain of overflow pages that is initially `NULL`. A hash function $h$ maps $A\rightarrow[0,...,N-1]$, with the result being the bucket in which the desired value is found.
![[w3n4chainedHashTable.png]]

If the overflow chain is not used, a search requires a single IO operation and a delete requires 2 IO operations.
## Hash collisions
Hash collisions are unavoidable, either through chance of two keys hashing to the same value or through search keys not being unique. This leads to overflow chains, and long overflow chains degrade performance as the entire chain may need scanned for each lookup. This means that $h$ should spread keys evenly across $[0,...,N-1]$, and with a large enough number of entries long chains will still occur.
## Dynamic files
As a file grows, the overflow chains spoil the index IO behaviour. If the file shrinks, a significant portion of the primary buckets may become (almost) empty, a waste of space. We could periodically rehash the index to restore the ideal situation (~20% free space and no overflow chains) but this is expensive and the index is unusable while rehashing is in progress.

Like ISAM, static hashing has advantages for concurrent access as only one bucket needs to be locked to store a new entry or extend the overflow chain.
# Extensible hashing
Instead of hashing directly into buckets, hash into a directory of pointers to buckets
![[w3n4extensibleHashing.png]]

We have a global depth $n$ (above it is 2), and the directory size is $2^n$, allowing for the $n$ least bits of $h(k)$ to be used to find a bucket pointer in the directory. Each bucket has a local depth $d,d<n$, where $h(k)$ of all entries in the bucket have the same least $d$ bits. For a bucket, the number of pointers to that bucket $=n-d+1$.

## Example
If we insert $B$ into the above hash table, first we find $h(B)=29=11101_2$, which tells us which bucket to insert into:
![[w3n4extensibleHashingEx1.png]]
The bucket has space, so we store $B$ in it. Next, we insert $C$, so $h(C)=5=00101_2$, so:
![[w3n4extensibleHashingEx2.png]]
except the bucket is full, so we increase the local depth and split it into two buckets ensuring that each maintain the same local depth, and increase the global depth:
![[w3n4extensibleHashingEx3.png]]
and insert $C$ into the new space:
![[w3n4extensibleHashingEx4.png]]

As we use the least significant bits for the directory, doubling the directory copies the previous pointers unchanged into the higher half, and just needs to fix the pointer for the new split.

If the local depth is less than the global depth for a bucket, then that bucket can be split without changing the global depth, only requiring modifying the directory pointers. 