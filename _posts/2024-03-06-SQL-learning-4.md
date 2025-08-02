---
title: SQL learning 4 - Multiple Tables
date: 2024-03-06 00:00:00 +0800
categories: [CheatSheet, SQL]
tags: [sql, learning]
math: true
# TAG names should always be lowercase
---
Update Date: 2024-03-06

## Multiple Tables
### Combine Tables by Inner Joins
```sql
SELECT table1.c1,
   table2.c2
FROM table1
JOIN table2
  ON table1.c3 = table2.c3;
```
This merges two tables according to `c3`.
If there are some unique values of column3 for a table, they would be removed.

### Left Joins
```sql
SELECT *
FROM table1
LEFT JOIN table2
  ON table1.c2 = table2.c2;
```
A left join will keep all rows from the first table, regardless of whether there is a matching row in the second table. If no matching, returns null for the corresponding rows.

### Primary Keys vs Foreign Keys
* Primary keys: the unique ID for a table
* Foreign keys: the ID (might be repeated) for connecting tables.

### Cross Join
```sql
SELECT table1.c1,
   table2.c2
FROM table1
CROSS JOIN table2;
```
This returns all possible combinations of the table. For example, if `table1.c1` has 3 rows and `table2.c2` has 2 rows, then this query returns 6 combinations (\$ 3 \times 2 $).

### Union
```sql
select * from table1
union
select * from table2
```
This combines of two tables, table1 over table2.

### With
```sql
with previousResults (
    SELECT *
    FROM table1
)
SELECT *
FROM previousResults
UNION
SELECT *
FROM table2;
```
`With` statement stores the result of a query in a temporary table (arbitrary alias like `previousResults` in the example). 