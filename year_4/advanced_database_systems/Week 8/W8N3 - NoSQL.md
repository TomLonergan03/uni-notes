NoSQL database systems were developed in the 2010s to handle high-volume [[W8N2 - Storage models#Database workloads|OLTP]] operations for massive-scale web applications (e.g. Facebook, Amazon). They trade-off reduced consistency and lack of transaction support for improved scalability, and usually have flexible schema.
# Scaling
## Scaling through partitioning
[[W8N1 - Parallel query processing#Database partitioning|Partitioning]] data across multiple machines allows for more clients to access the database simultaneously, and can make writes more efficient, at the cost of expensive reads that may require accessing many machines, and leads to concurrency challenges.
## Scaling through replication
Replicating databases across machines and distributing queries among replicas allows for more simultaneous clients, improved fault tolerance, and efficient reads, at the cost of expensive writes (as all replicas must be updated), and difficulties maintaining consistency.
# NoSQL paradigm (BASE)
- **Basically available**: availability is guaranteed even during failures, but responses may not contain fully consistent data.
- **Soft state**: the state of the system can change over time, even without updates, as replicas may not have synchronised with the rest of the system yet.
- **Eventually consistent**: data will eventually become consistent across all nodes, though there is no guarantee of immediate consistency.
# Kinds of NoSQL systems
## Key-value stores
**Key-value (KV) stores** store data as key value pairs, with the only operations permitted being `get(key)` to get a value, or `put(key, value)` to insert a value if `key` doesn't exist, or update a value if `key` does exist. Partitioning stores a key $k$ at server $h(k)$, and multiway replication stores a key $k$ at servers $h1(k),h2(k),h3(k)$. Examples include Redis, DynamoDB, and Memcached, and KV stores are used for caching, session storage, and simple data storage.
## Document stores
**Document stores** store data as JSON, BSON, XML, etc. documents. Examples include MongoDB and CouchDB, and use-cases include content management, e-commerce, and user profiles.
## Column-family stores
Data is stored as columns and column families. These are used for large-scale analytics, time-series data, and log processing, and examples include Cassandra and HBase/Google BigTable.
## Graph databases
**Graph databases** store data as nodes and edges (relationships between nodes). Examples include Neo4j, ArangoDB, and Amazon Neptune, and use-cases include social networks, recommendation engines, and fraud detection.