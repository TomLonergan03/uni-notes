Most database systems are designed for non-volatile disk storage, which requires data to be moved into main memory for processing. Disk reads and writes are very slow, so accesses must be planned very carefully, and data stored on disk is not byte addressable, so it is transferred in pages.
For an overview of storage devices, see [[W7N1 - Secondary storage|here]].
# Sequential vs random access
If we use an example disk of the Seagate Cheetah 15k.7, it has the following stats:

| Stat             | Value     |
| ---------------- | --------- |
| Disks            | 4         |
| Heads            | 8         |
| Avg. track size  | 512 KB    |
| Capacity         | 600 GB    |
| Rotational speed | 15000 rpm |
| Avg. seek time   | 3.4 ms    |
| Transfer rate    | 163 MB/s  |
The average access time to read one 8 KB block is:
$$
\begin{aligned}
\text{Average seek time}&=3.40ms\\
\text{Average rotational delay}&=\frac{1}{2}\cdot\frac{1}{15000}\min=2.00ms\\
\text{Transfer time}&=\frac{8KB}{163MB/s}=0.05ms\\
\text{Total access time}&=\text{seek time}+\text{rotational delay}+\text{transfer time}=5.45ms
\end{aligned}
$$
This shows that seek time and rotational delay dominate access time.

A series of sequential access are much faster than random accesses, as we do not have to reseek inbetween each read. If we read 1000 8 KB blocks, it will take:
- **Random**: $1000\cdot5.45ms=5.45s$
- **Sequential**: $3.4ms+2ms+1000*.05ms\approx55ms$ (with some additional (<5ms) track to track seek times)

This means that a DBMS should avoid random disk accesses at all costs.
## Arranging blocks on disk
When we arrange blocks on a disk, we want to maximise the chance that the next block read is:
1. A sequential block on the same track
2. If not, then a block on the same cylinder
3. If not, then on an adjacent cylinder

For a sequential scan, we can prefetch several blocks at a time, as reading large consecutive blocks amortises the seek and rotational delays over all blocks read.
## Solid state drives
Solid state drives have consistent reads across both access patterns:
- Single read access time $= 30 \mu s$
- 4 KB random reads $\approx500MB/s$
- Sequential reads $\approx525MB/s$

But writes are not:
- Single write access time$=30\mu s$
- 4KB random writes $\approx 120MB/s$
- Sequential writes $\approx480MB/s$
# Overall
Very large databases are relatively traditional and rely mostly on HDDs, with SSDs serving as caches to improve performance. Smaller databases can often use SSDs, and many small databases can be fully stored in RAM. Non-volatile memory is likely to affect the design of future systems.

For the purposes of this course we will focus on traditional RAM and disk.