# Cost model
To compare file organisation's impact on performance, we will use a simple **IO only** cost model that is good enough to show the overall trends. This model has:
$$
\begin{matrix}
P:&\text{Number of data pages in the file}&=5\\
R:&\text{Number of records per page}&=2\\
D:&\text{Average time (ms) to read or write a disk page}&=5
\end{matrix}
$$
For now, we will ignore sequential vs random IO, page prefetching, and any CPU costs.
# Record operations
There are a number of different record operations we may want to perform:
- Scanning all records in a given file: `SELECT * FROM R`
- Searching for records with an equality test: `SELECT * FROM $ WHERE C = 42`, both on key attributes (where there is exactly one match), and non-key attributes (where there may be multiple matches)
- Searching for records that fall in a range: `SELECT * FROM R WHERE A > 0 AND A < 100`
- Single record inserts and deletes, with heap files having inserts appended to end of file, and sorted files performing compactions after deletions.
# Heap files
Heap files contain records in no particular order.
## Scan
Read each page of the file for every page scan over all records. Scanning all records touches $P$ pages, and reading each page takes $D$ time, for an estimated cost of $P\cdot D\ ms$.
## Search on key
If we assume there is exactly one match, and that the probability of the key being on any page is equal ($1/P$), then if the key is on page $i$ we will need to read $i$ pages. This results in an expected number of touched pages:
$$
\begin{aligned}
\sum^{P}_{i=1}i\frac{1}{P}&=\frac{1}{P}\frac{P(P+1)}{2}\\
&=\frac{P+1}{2}\\
&\approx\frac{P}{2}
\end{aligned}
$$
for an estimated cost of $P/2\cdot D\ ms$.
## Search on non-key
We have to check all pages, so has an estimated cost of $P\cdot D\ ms$.
## Range search
We have to check all pages, so has an estimated cost of $P\cdot D\ ms$.
## Insert
We have to read the last page, append a new record, and write it back to disk, for a cost of $2D\ ms$.
## Delete
This consists of a search on key, followed by deleting the record from the page and writing it back to disk, for a cost of $(P/2 + 1)\cdot D\ ms$.
# Sorted files
Sorted files store records sorted by lookup attributes with no gaps.
## Scan
Again, all pages must be read, so cost is $P\cdot D\ ms$.
## Search on key
We can use a binary search to find the key, which costs $\log_2(P)\cdot D\ ms$.
## Search on non-key and range
We can use binary search for the start of the range, then scan rightwards to find the rest of the matching records, for a cost of $(\log_2(P)+\text{number of pages with matching records})\cdot D\ ms$.
## Insert and delete
An insert and delete will requiring finding the insertion/deletion point, then shifting the rest of the file after the insertion or deletion. On average this will shift $P/2$ files, with a read and a write each, resulting in a total cost of $\log_2(P)\cdot D+P\cdot D\ ms$.
# Indexes
[[W3N2 - Indexes|Index files]] allow for fast lookup by a specific value.