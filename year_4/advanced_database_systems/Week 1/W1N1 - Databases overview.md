# What is a database
A **database** is a collection of inter-related data. A **database management system (DBMS)** is software that stores, manages, and facilitates access to a database.
# Flat files
One of the simplest ways to store a database is in a collection of comma-separated value (CSV) files, e.g.
```
instructors.csv
========================
name, dept, salary
“Jones”, “CS”, 95000
“Smith”, “Physics”, 75000
“Gold”, “CS”, 62000
```
```
courses.csv
========================
name, instructor, year
“Databases”, “Jones”, 2018
“Quantum M.”, “Smith”, 2017
“Compilers”, “Jones”, 2017
```
This approach has many drawbacks:
- **Data redundancy**: the same data may be duplicated in many different files, which makes updates complicated and error prone
- **Exposed storage format**: an application developer needs to be aware of the physical layout of the data
- **Difficult access**: a new program needs to be written for each new task, and programming complex logic on several files can be error prone and inefficient
- **Expensive searches**: the only way to find a record with a specific key is by reading the entire file and checking every record in turn
- **No atomicity of updates**: if a failure occurs part way through an update, the database may be left in an inconsistent state with some parts updated while others haven't been
- **Integrity issues**: integrity constraints (e.g. course mark must be $\geq$ 0) are buried in the program code, with it being hard to add new constraints or change existing ones
- **Concurrency problems**: if simultaneous accesses are needed for multiple users or performance, they are complex to support and can lead to inconsistencies
- **No security**: it's difficult to provide a user access to some but not all data
- **No API**: there is no programmatic way to access the data other than parsing the entire file, making it difficult to integrate it with other programs.

All of these issues are solved in some way by DBMSes.