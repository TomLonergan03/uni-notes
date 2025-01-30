A tree-based [[W3N2 - Indexes|index]] is designed to allow for faster searches, as the index file will be much smaller than the size of the data. The index file may still be quite large, so it can itself be indexed, which is repeated until the topmost index level fits on one page (the **root page**)
# ISAM
![[w3n3isam.png]]
Under **ISAM (indexed sequential access method)**, non-leaf pages direct searches to their children, while the leaf pages contain sorted index data entries. Overflow pages then contain additional data that is inserted into the system. ISAM is a static tree, meaning that inserts and deletes only affect leaf pages.
## Usage
- **Search**: start at the root, and use key comparisons to find correct child node. Repeat until a leaf is reached.
- **Insert**: find the leaf for the new record, and insert it if there is space. If there isn't, create an overflow page hanging off of the primary page
- **Delete**: find the record in a leaf and delete the record. If an overflow page becomes empty, deallocate it
## Example
If we start with this tree:
![[w3n3isamEx1.png]]
then insert `23*`, `48*`, `41*`, and `42*`, we get:
![[w3n3isamEx2.png]]
then delete `42*`, `97*`, and `51*`, we get:
![[w3n3isamEx3.png]]
## ISAM performance
- As non-leaf levels aren't changed during inserts/deletes, the tree does not need to be locked during concurrent index accesses.
- With heavy updating, the tree may lose balance with long chains of unsorted overflow pages being created over time. This causes gradual degradation of search performance
- ISAM is optimal for relatively static data
## ISAM vs binary search
If we have:
- $N$ pages in the data file
- A maximum fanout (number of children) $F$ per index node
	- $F=3$ in the above example, $F\approx1000$ typically

Then from the root page we choose an index subtree of size $N/F$. After $s$ steps down the tree, the search space is reduced to $N\cdot(1/F)^s$. Assuming it takes $s$ steps to reach a leaf, then $N\cdot(1/F)^s=1$. hence $s=\log_F(N)$. If $F>2$, then $\log_F(N)<\log_2(N)$, so ISAM will outperform binary search.
# B+ tree
A **B+ tree** is similar to ISAM but:
- Has no overflow chains, so search performance is only dependent on height, and due to a high fanout the height rarely exceeds 3
- Offers efficient insert/delete procedures
- Every node (except the root) has a minimum occupancy of 50%.
- Has an order $d$, with each node containing $d\leq m\leq2d$ entries

A B+ tree of order 2 looks like:
![[w3n3b+tree.png]]
Each non-root node contains $2\leq m\leq4$, for a maximum fan-out of $2d+1$.
## In practice
- Typical order $=100$, with a typical fill-factor of 67%, resulting in a fanout $F=2*100*0.67=133$
- Typical capacities:
	- Height 3: $133^3=2,352,637\text{ records}$
	- Height 4: $133^4=312,900,721\text{ records}$
	- Height 5: $133^5=41,615,795,893\text{ records}$
- Top level pages will often fit in the buffer pool:
	- Level 1 has a size of $1\text{ page}=8KB$
	- Level 2 $=133\text{ pages}=1MB$
	- Level 3 $=17,689\text{ pages}=138MB$
## Usage
### Lookup
Lookup is the same as in an ISAM tree.
### Insert
First, find the correct leaf $L$, insert entry into $L$ in sorted order. If $L$ has enough space, the insert is done, otherwise split $L$ into $L$ and $L_2$ with half the entries in each, and copy up the middle key to $L$'s parent, and add an index entry pointing to $L_2$ 
![[w3n3b+Insert.png]]
As 5 is pushed to the root, the root is then split, and a new root is created that only contains 17 and the pointer to each of the two halves of the previous root:
![[w3n3b+InsertPt2.png]]
If we now insert 6, instead of splitting the page we can *redistribute*, and push 7 to the sibling node, along with updating the parent:
![[w3n3b+InsertPt3.png]]This is possible as there is space in the target node's direct sibling.
### Delete
First, find leaf $L$ where the entry exists and remove the entry. If $L$ is still at least half full, the delete is done. If not, first try to redistribute an entry from a sibling, and update the parent value. If that fails, merge $L$ and its sibling, and delete the entry and pointer from the parent of $L$. This may propagate up the tree, and if it reaches the root it will decrease the height of the tree.

If we delete 19 from the example tree, no underflow occurs so no more work is needed:
![[w3n3b+DeleteEx1.png]]
then when we delete 20 we can redistribute from its sibling:
![[w3n3b+DeleteEx2.png]]
but it is more complex when we delete 20. First, the leaf nodes $p$ and $p^\prime$  merge:
![[w3n3b+DeleteEx3.png]]
then the new pages $p$ and $p^\prime$ merge by pulling down the separator, along with the root being deleted:
![[w3n3b+DeleteEx4.png]]

Alternatively, if the left subtree has a different structure:
![[w3n3b+DeleteEx5.png]]
$p$ and $p^\prime$ are redistributed by pushing 20 up to the root and 22 down to $p$:
![[w3n3b+DeleteEx6.png]]

In practice, the occupancy invariant is often not maintained with nodes only being deleted if the node is completely empty.
## Variable length keys and records
So far we have been using integer keys, but we may want to use variable length keys:
![[w3n3varLengthKeys.png]]
or store data in leaf pages directly using [[W3N2 - Indexes#^055a12|variant A]]:
![[w3n3variantA.png]]
We can instead require that a node is at least half full in bytes, as non-leaf pages will now often hold many more entries than leaf pages.
## Optimisations
### Prefix compression
As leaf level keys are sorted, they are likely to have the same prefix:
![[w3n3varLengthKeys.png]]
Here, all keys start with rob, so can be stored as:
![[w3n3prefixCompression.png]]
### Suffix truncation
Non-leaf level nodes only need to store enough information to direct a request in the correct direction, so can be compressed from:
![[w3n3suffixTruncationBefore.png]]
to:
![[w3n3suffixTruncationAfter.png]]
