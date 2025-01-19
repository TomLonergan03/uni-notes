**SQL (Structured Query Language)** is a declarative language used for querying relational databases.
# Relational databases
**Databases** are a set of named relations. **Relations/tables** have a **schema** (a description of how data should be laid out, e.g. `Student(sid: int, name: text, dept: text)`) and zero or more **instances** (collections of data satisfying the schema) 
![[w1n2relations.png]]

The schema is fixed, and must have unique attribute names. Attribute types are atomic. Instances change regularly as writes occur.
# SQL language features
SQL consists of three sub-languages:
- **DDL (data definition language)** is used to define and modify schemas
- **DML (data manipulation language)** is used to write queries
- **DCL (data control langauge)** is used to control access to data

The RDBMS is responsible for efficient evaluation, so the algorithms used for queries may change, though the choice of algorithm must not affect the query answer.
# Example database
We will use a simple example database for the purposes of the topic, which has three tables:

`Student(sid: int, name: text, dept: text, age: int)`

| sid   | name  | dept    | age |
| ----- | ----- | ------- | --- |
| 12344 | Jones | CS      | 18  |
| 12355 | Smith | Physics | 23  |
| 12366 | Gold  | CS age  | 21  |
| ...   |       |         |     |

`Course(cid: text, name: text, year: int)`

| cid       | name                        | year |
| --------- | --------------------------- | ---- |
| INF-11199 | Advanced Database Systems   | 2020 |
| INF-10080 | Introduction to Databases   | 2020 |
| INF-11122 | Foundations of Databases    | 2019 |
| INF-11007 | Data Mining and Exploration | 2019 |

`Enrolled(sid: int, cid: text, grade: int)`

| sid   | cid       | grade |
| ----- | --------- | ----- |
| 12344 | INF-10080 | 65    |
| 12355 | INF-11199 | 72    |
| 12355 | INF-11122 | 61    |
| 12366 | INF-10080 | 80    |
| 12344 | INF-11199 | 53    |
| ...   |           |       |
# Basic single-table queries
## SELECT FROM
```sql
SELECT [DISTINCT] <column expression list>
	FROM <single table>
	[WHERE <predicate>]
```
This is straightforward, the RDBMS will produce all tuples in the table that match the predicate, then output the expressions in the `SELECT` list.

e.g. to get all information about all 18 year old students:
```sql
SELECT * FROM Student WHERE age = 18
```
## ORDER BY
A `SELECT` statement can be sorted using `ORDER BY`:
```sql
SELECT sid, grade FROM Enrolled
WHERE cid = ‘INF-11199’
ORDER BY grade
```
Sort order is ascending by default, but can be overridden:
```sql
SELECT sid, grade FROM Enrolled
WHERE cid = ‘INF-11199’
ORDER BY grade DESC, sid ASC
```
## LIMIT
The number of tuples returned can be limited using `LIMIT <count> [offset]`:
```sql
SELECT sid, grade FROM Enrolled
WHERE cid = ‘INF-11199’
ORDER BY grade LIMIT 3 OFFSET 1
```
This will return the 2nd, 3rd, and 4th lowest grades in the class.
This is generally used with `ORDER BY`, as without it the result is nondeterministic.
## Aggregates
Aggregates apply an arithmetic expression to a returned set of records. Aggregates include `SUM`, `COUNT`, `MIN`, `MAX`, `AVG`.
```sql
SELECT AVG(age) AS avg_age FROM Student Where dept = 'CS'
```
This will return the average age of all students in the CS department.
## GROUP BY#
`GRUP BY` will partition a table into groups based on a column value. These groups can then be aggregated.
```sql
SELECT dept, AVG(age) AS avg_age FROM Student GROUP BY dept
```
This will return the average age of students in each department.
All non-aggregated values in the `SELECT` clause much appear in the `GROUP BY` clause.
## HAVING
`HAVING` filters results after grouping and aggregation. 
```sql
SELECT dept, AVG(age) AS avg_age FROM Student GROUP BY dept HAVING AVG(age) > 21
```
This returns all departments where the average student age is greater than 21.
## Conceptual evaluation order
![[w1n2sqlEvaluationOrder.png]]
A query is evaluated so that the result is formed as if it was evaluated in this order. In practice, these steps may be reordered depending on the algorithm used by the DBMS.
# Multi-table queries
Join queries join two tables by matching values in multiple tables. It takes the form:
```sql
SELECT <column list>
FROM <table>
[INNER | NATURAL | {LEFT | RIGHT | FULL} OUTER] JOIN 
ON <qualification list>
WHERE ...
```
## INNER JOIN
An `INNER JOIN` returns all records where both the left and right side of the join is present.
e.g. with these two tables
`names`

| key | name    |
| --- | ------- |
| 1   | Alice   |
| 2   | Bob     |
| 3   | Mallory |
`ages`

| key | age |
| --- | --- |
| 1   | 25  |
| 2   | 30  |
| 4   | 51  |

running this query:
```sql
SELECT name, age
FROM names INNER JOIN ages
ON names.key = ages.key
```
will return:

| name  | age |
| ----- | --- |
| Alice | 25  |
| Bob   | 30  |
## NATURAL JOIN
A `NATURAL JOIN` will perform an inner join on two tables which share only one common column. E.g. reusing the `names` and `ages` tables, we can run
```sql
SELECT name, age
FROM names NATURAL JOIN ages
```
will produce:

| name  | age |
| ----- | --- |
| Alice | 25  |
| Bob   | 30  |
## LEFT OUTER JOIN
A `LEFT OUTER JOIN` will return all matched rows and preserve all unmatched rows on the left side of the `JOIN` clause, replacing unmatched fields with `NULL`.
E.g. running
```sql
SELECT name, age
FROM names LEFT OUTER JOIN ages
ON names.key = ages.key
```
will produce:

| name    | age  |
| ------- | ---- |
| Alice   | 25   |
| Bob     | 30   |
| Mallory | NULL |
## RIGHT OUTER JOIN
A `RIGHT OUTER JOIN` will do the same as a `LEFT OUTER JOIN` but preserve the right-hand unmatched rows.
E.g.
```sql
SELECT name, age
FROM names RIGHT OUTER JOIN ages
ON names.key = ages.key
```
will produce:

| name  | age |
| ----- | --- |
| Alice | 25  |
| Bob   | 30  |
| NULL  | 51  |
## FULL OUTER JOIN
A `FULL OUTER JOIN` is the same as the other joins, except it preserves unmatched rows from both sides of the join clause.
E.g.
```sql
SELECT name, age
FROM names FULL OUTER JOIN ages
ON names.key = ages.key
```
will produce:

| name    | age  |
| ------- | ---- |
| Alice   | 25   |
| Bob     | 30   |
| Mallory | NULL |
| NULL    | 51   |
# Nested queries
Queries can contain other queries, which can appear almost anywhere in a query. They are often difficult for the RDBMS to optimise.
```sql
SELECT S.name FROM Student S WHERE S.sid IN ( SELECT E.sid FROM Enrolled E )
```
## Set comparison operators
There are a number of set comparison operators, including:
- `<op> ALL` - satisfies expression for all rows in subquery, where op is a standard comparison operator
- `<op> ANY`- satisfies expression for at least one row in subquery
- `IN` - equivalent to `= ANY (...)`
- `NOT IN` - equivalent to `!= ANY (...)`
- `EXISTS` - at least one row is returned

