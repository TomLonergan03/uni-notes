# Database workloads
There are two main types of database workloads:
- **On-line transactional processing (OLTP)**: many users performing simple operations that handle small amounts of data per transaction. These tend to be for operational tasks, such as order processing or payments. They tend to read and update a small amount of data related to a single entity.
- **On-line analytical processing (OLAP)**: small numbers of complex queries that read large amounts of data to compute aggregates. These tend to be for data analysis/reporting, such as getting business insights from historical data. They tend to be long running, read heavy, and with many aggregations.

There are also now **hybrid transactional and analytical (HTAP)** workloads, which have OLTP and OLAP workloads running on the same database instance, such as real-time analytics for fraud detection.
# Storage models
The **storage model** specifies how tuples are physically arranged on disk and in memory. Different storage models can have different performance characteristics depending on the target workload.
## Row storage
![[w8n2rowStorage.png]]
The **row storage model** stores all attributes of a tuple contiguously in memory and on disk. This is beneficial for OLTP workloads as they frequently insert/update/delete single tuples.

On disk, this is stored using slotted pages, with a record id identifying a page and a slot. An index can also directly point to an entire tuple on disk.
![[w8n2rowStorageIndex.png]]

If we perform an OLAP workload, we will often read a lot of data we don't use:
![[w8n2rowStorageOlap.png]]
### Advantages
- Fast access to all attributes of a single tuple.
- Ideal for OLTP workloads involving individual tuple operations.
- Can use clustered indices in [[W3N2 - Indexes#^055a12|variant A]] for storing data.
### Disadvantages
- Entire rows must be read even if a query only involves a few attributes.
- Poor memory locality as the same attribute in two tuples is separated by all the rest of the data in that tuple.
- Multiple value types on the same page make compression less efficient.
## Columnar storage
![[w8n2columnStorage.png]]
With **columnar storage**, each attribute for all tuples is stored contiguously. Attributes and metadata are stored in separate arrays of fixed length values, and physical tuples are identified using offsets into these arrays.

![[w8n2columnStorageOlap.png]]
Now, we only read the exact attributes needed for an analytical query, reducing the wasted IO time.
### Advantages
- Reduces wasted IO, as it provides free [[W5N2 - Query optimisation#Projections|projection pushdown]].
- Faster query processing because of increased cache locality.
- Data compression is more effective as every value in a page are in the same domain.
### Disadvantages
- Slow for point queries, inserts, updates, and deletes as tuples need split across pages and stitched back together on read.
## Hybrid storage (PAX)
OLAP workloads rarely access only a single column in isolation. Ideally, we want the columnar benefits of compression and efficient processing without losing the speed of accessing related data together. **Partition attributes across (PAX)** is a hybrid storage model that vertically partitions attributes within a page. Examples include Parquet, ORC, and Arrow.

Data is horizontally partitioned into row groups, then each row group is partitioned into column chunks. A global metadata directory contains offsets to the file's row groups, which is stored in the footer if the file is immutable (e.g. Parquet, Orc).
![[w8n2hybridStorage.png]]
### Parquet file format
![[w8n2parquet.png]]
Parquet splits rows into row groups, by default 128MB each, then splits those groups into column chunks. A column chunk is divided into pages, by default 1MB each, which contain metadata such as precomputed aggregates (min/max/count), and repetition and definition information for nested data, along with compressed data values.
The footer then contains file, row, and column metadata, such as schema, count, and row group offsets.
# Compression
Compressing files will reduce the storage and RAM requirements, and increases the amount of data read per IO. Database compression must be lossless, with the application performing lossy compression should it choose to. The key trade-off is compression ratio vs speed, equivalent to lower IO vs higher CPU costs.
## Naive compression
General purpose compression algorithms compress data block by block with no semantic understanding, and require decompression before reading or modification. The limited scope leads to lower compression ratios on heterogeneous data.
## Columnar compression
There are a number of approaches to compress columnar data:
- **Run-length encoding**: remove duplicates by storing data as `<value, number of occurrences>`. This is good for mostly sorted integers or categorical data. E.g. $2,2,2,3,4,4,4,4,4\rightarrow2\times3,3\times1,4\times5$.
- **Delta encoding**: encode runs of values using the difference between subsequent values, e.g. $2,3,4,5\rightarrow2,+1,+1,+1$. It is good for mostly sorted numeric data, and can pair well with run length encoding.
- **Bit packing**: use fewer bits for short integers. It is good for limited precision data, and pairs well with delta encoding.
- **Dictionary encoding**: replace frequently used values with smaller fixed-length codes, and maintain a mapping from the codes to the original values. It is good for long, frequent strings and categories. Some queries don't require converting each code back to a value, saving space. ![[w8n2dictionaryEncoding.png]]
### Delta encoding in Parquet
Parquet encodes numbers using delta encoding. 

If we have a series of values 
$$
100,101,101,102,101,101,102,101,99,100,105,107,114,116,119,120,121
$$
Then we first take a reference (the first value) and split the remainder into blocks of 64 bytes:
$$
\begin{matrix}
\text{Reference}&\text{Block 1}&\text{Block 2}\\
100&101,101,102,101,101,102,101,99&100,105,107,114,116,119,120,121
\end{matrix}
$$
Then convert each value to the delta between that entry and the previous one:
$$
\begin{matrix}
\text{Reference}&\text{Block 1}&\text{Block 2}\\
100&1,0,1,-1,0,1,-1,-2&1,5,2,7,2,3,1,1\\
\end{matrix}
$$
Then we find the minimum delta for each block, store it, and subtract it from each delta in that block:
$$
\begin{matrix}
\text{Reference}&\text{Min delta 1}&\text{Block 1}&\text{Min delta 2}&\text{Block 2}\\
100&-2&3,2,3,1,2,3,1,0&1&0,4,1,6,1,2,0,0\\
\end{matrix}
$$
The binary encoding of the blocks are as follows:
$$
\begin{matrix}
\text{Block 1}&\text{Block 2}\\
11,10,11,01,10,11,01,00&000,100,001,110,001,010,000,000\\
\end{matrix}
$$
We can then see that block 1 requires 2 bits per delta, while block 2 requires 3 bits per delta. These values are stored, and the bits are packed together:
$$
\begin{matrix}
\text{Reference}&\text{Min delta 1}&\text{Bit length 1}&\text{Packed value 1}\\
100&-2&2&1110110110110100\\
&\text{Min delta 2}&\text{Bit length 2}&\text{Packed value 2}\\
&1&3&000100001110001010000000
\end{matrix}
$$
This is all the information we need to store. The initial series was $17\cdot8=136\text{ bytes}$, while the final result is $8+8+1+2+8+1+3=31\text{ bytes}$ (as the bit length can always fit in one byte assuming no larger than 64 bit numbers).