DBMSes tend to be designed in a number of layers, with each layer talking to the layers directly above and below it.
![[w2n1dmbsArchitecture.png]]
1. **Query planning**: a query is parsed, checked, verified, and translated into an efficient relational query plan
2. **Operator execution**: a dataflow is executed by operating on files and records
3. **Files and index management**: organises tables and records as groups of pages in a logical file
4. **Buffer management**: determines when data is transferred between disk and memory
5. **Disk space management**: translates page requests into reading and writing physical bytes on devices

There are also two cross-cutting modules:
- **Concurrency control**: manages concurrent connections to the database
- **Recovery**: manages recovery of the database following a crash, ensuring a consistent view of the database

