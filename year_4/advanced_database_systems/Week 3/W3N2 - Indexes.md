**Index files** allow for fast lookup by a specific value. An index is created on any subset of attributes in a relation:
```SQL
CREATE INDEX idx1 ON Student USING btree(sid);
CREATE INDEX idx2 ON Student USING hash(sid);
CREATE INDEX idx3 ON Student USING btree(age, name);
```
Indexes speed up searches, at the cost of adding overhead for inserting, updating, and deleting records.

![[w3n2index.png]]
An index entry $k^*$ consists of the key $k$, and some other data:
![[w3n2indexEntries.png]]
- Variant A stores the entire record in the index file. To avoid redundant records, at most one index on a table can use this method. ^055a12
- Variants B and C use record IDs (`rid`s) to point to the record in a heap file.
# Index classification
Indexes can be:
- **Tree-based** vs **hash-based**: some operations are only possible on one type, e.g. range search only works with tree indexes.
- **Clustered** vs **unclustered**: in a clustered index records are ordered similarly to how they are ordered in the heap file.
- **Primary** vs **secondary**: a primary index is built on a primary key, a secondary index is built on a non-key attribute.
## Clustered vs unclustered indexes
![[w3n2clusteredVsUnclustered.png]]
A clustered index is built on a sorted hep file, which has some free space (usually ~1/3rd) left for future inserts. The cost of retrieving records from the index varies greatly: for clustered it is approximately the number of pages in the data file with matching records, while for unclustered is is approximately the number of matching records.

Clustered indexes are efficient for range searches, have potential for locality allowing for sequential disk accesses and prefetching, and support certain types of compression as sorted data has a high likelihood of repetitive patterns that compression algorithms can exploit.

Clustered indexes are more expensive to maintain, as the heap file must be periodically updated, either on the fly or as a separate process, and not packing the heap files will waste space on disk.