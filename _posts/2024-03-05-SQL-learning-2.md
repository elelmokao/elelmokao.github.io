---
title: SQL learning 2 - Queries
date: 2024-03-05 00:00:00 +0800
categories: [CheatSheet, SQL]
tags: [sql, learning]
# TAG names should always be lowercase
---
Update Date: 2024-03-05

## Query Statement
### Select 
```sql
-- select all columns
select * from tableName;
-- select specific columns
select column1, column2, ..., columnN from tableName;
```
### As
```sql
-- rename column
select column1 as 'columnA',
  column2 as 'columnB',
  column3 as 'columnC'
from tableName

```
### Distinct
```sql
select distinct column1 from tableName;
```
This returns all unique values in column1. 

### Where
```sql
select column1 from tableName
where condition
-- like year = 2017; year < 2017; year > 2017
```

### Like & Between
```sql
-- like
select * from tableName
where column1 like pattern
```
* pattern: a string composed of '%' (matches zero to any number of arbitrary characters) or '_' (maches a single arbitrary characters)
* e.g. '_H%' matches 'AHEAD', 'WHERE', 'THE' but not "HOUSE" or "BREATH"

```sql
-- between
select * from tableName
where column1 between value1 and value2
```
This returns rows that column1 >= value1 and column1 <= value2.
`between` can still be used for characters:
```sql
select * from tableName
where column1 between char1 and char2
```
This returns rows that column1's string starts with char1 and column1's string starts with char2.

For example:
If char1 = 'A' and char2 = 'B'. This returns rows with column1 starting with 'A' (A, Absorb, A..., Azure) and end with 'B' ('B' only). The words starting with 'B' will not be included but 'B'.

### And & Or
```sql
-- And
select * from tableName
where condition1 and condition2
-- Or
where condition1 or condition2
```

### ORDER BY
```sql
-- in an ascending order (default)
select * from tableName
order by column1 ASC
-- in a descending order
select * from tableName
order by column1 DESC
```
`ORDER BY` always goes after `WHERE` (if `WHERE` is present).

### LIMIT
```sql
select * from tableName
LIMIT value
```
This shows only [value] rows. `limit` should always goes at the very end of query and not supported by all SQL databases.

### CASE
```sql
select *
case
  when case conditionA then a
  when case conditionB then b
  else c
end -- as columnName
from tableName
```
Add `as columnName` can rename these a, b, c strings to a new column name.