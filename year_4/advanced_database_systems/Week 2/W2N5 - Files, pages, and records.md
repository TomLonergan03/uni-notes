![[w2n5fileAndIndexManagement.png]]
![[w2n5tableRepresentations.png]]
Tables are stored in the database file, which is divided into pages. Each page consists of one or more records. All of these must be represented on the disk.

The file manager gives an API to higher levels of the DBMS:
- Create/delete a file
- Insert/modify/delete a record
- Fetch a particular record by record ID
- Scan all records, possibly with a predicate on desirable records
# File organisation
There are several ways to arrange records into files, with each being used in certain situations.
- Heap files place records arbitrarily across pages
- Sorted files have pages and records in sorted order
- Index files use B+ trees or hash-based files, and may contain records or point to records in other files
## Heap files
**Heap files** contain a collection of records in no particular order. To support record level operations, we must keep track of the pages in a file, the free space on pages, and the records on a page.

There are two basic organisations:
- Doubly linked lists of pages:
	- A header page contains two lists of pages, one for full pages and one for pages with free space. Each page keeps track of free space in itself.
	- This is easy to implement, but most pages end up in the free space list which means that finding a page with sufficient empty space may require searching many pages, each of which may be an expensive disk read.
- Page directories:
	- The directory is a set of special pages storing metadata about data pages. Each directory entry identifies a page and the number of free bytes in it. This makes free space searches much more efficient, as only a few directory pages need to be read to find a page that will fit a record. The header pages are accessed often, so are likely to be in the cache.
# Page layout
![[w2n5pageLayout.png]]
Pages have a header, containing a page ID, number of records, free space, next/previous pointers, etc. and allow finding a record by record ID, inserting, and deleting records.
## Fixed length records
If all records are the same size, we can divide the page into slots and place each record into a slot.
## Fixed length packed records
![[w2n5fixedLengthPacked.png]]
We can guarantee no gaps between records by rearranging them on record deletion. This makes it easy to compute the offset of a record in a page (`sizeof(header) + (position - 1) * sizeof(record)`), and easy to insert a new record (`sizeof(header) + number_of_records *+* sizeof(record)`). This comes at the cost of having to move the last record into the empty slot, which changes its ID and may require updating record pointers in other files.
### Fixed length unpacked records
If we allow gaps between records, we need to keep track of where each record is. This can be done using a bitmap, where there is 1 bit for each slot that is set when that slot is in use, or using a free list where the header points to the first free slot, which then points to the next and so on.
## Variable length records
If fields in a record do not have a consistent length, we need to keep track of the offset of each record.
### Slotted pages
![[w2n5slottedPages.png]]
The most common layout scheme is **slotted pages**. A slot directory maps slots to the record's starting position offset, and records are stored at the end of the page. On deletes, we set the slot offset to -1 and only delete it if it is the last slot, and either fill the empty space from the record immediately, or periodically defragment the space. On insert, it finds a slot with offset -1 or creates a slot if there are none available, then allocates exactly the right amount of space. If there is not enough contiguous space available the page can be defragmented.
# Record layout
We don't need to store schema information with each record, as it is in the system catalogue. Some metadata is required, such as a bitmap stating which values are `NULL`, as there is no way to distinguish e.g. the empty string "" from a null string.
## Fixed length records
![[w2n5fixedLengthRecord.png]]
If every field in a record has a fixed length, we can access a field just by adding the field offset to the base address.
## Variable length records
If any field in the record can have variable length, they can either be deliminated by special symbols, requiring a scan of the record and those symbols to be escaped in fields:
![[w2n5varRecordSymbols.png]]
or having an array of fixed length offsets followed by the data:
![[w2n5varRecordFieldOffsets.png]]
An implementation might have fields stored in order, with variable-length fields represented by a fixed size `(offset, length)`, with the actual data stored after all fixed length data so this record:
![[w2n5varLengthRecordSchema.png]]
would be represented as this:
![[w2n5varLenRecordsImplementation.png]]
