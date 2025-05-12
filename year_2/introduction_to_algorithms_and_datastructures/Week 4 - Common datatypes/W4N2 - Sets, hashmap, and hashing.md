A set contains a number of unique values, and has the methods **contains**, **insert**, **delete**, and **isEmpty**.
A hashmap maps keys to values, and has the methods **lookup**, **insert**, **delete**, and **isEmpty**.

These are both most efficiently implemented as hash tables. In a set, we insert an item by hashing its value, and insert it at that location. Likewise in a hashmap we hash the key and insert the value at that location.

We encounter an issue should there be 2 keys with the same hash. In this case there are 3 main options:
- Each hash has a value which is a reference to a list.
	- In the worst case, every key hashes to the same location and we now have a list with the additional overhead of hashing
- We use the hash as a start point for probing.
- We use a perfect hash function which maps every key to exactly one location.
	- This requires foreknowledge of the values that can be used as keys when designing the hashing function and that there are a finite set of keys.

## Probing
When a hash is produced, we go that that location in the table and check it has the key we are looking for (in case of lookup), if it does then return it, otherwise look in the next location in the table. If an empty location is found, or it returns to the original location, then the key is not in the table.
This is good, as then the expected number of probes for insert and lookup stay low until the table is nearly full.
However, delete is difficult, as it must both move keys that do hash to the same value forward while also not doing so for something that was originally inserted at that location before the table filled up, and these values may be interleaved.