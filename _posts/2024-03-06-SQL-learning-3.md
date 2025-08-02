---
title: SQL learning 3 - Aggregation
date: 2024-03-06 00:00:00 +0800
categories: [CheatSheet, SQL]
tags: [sql, learning]
# TAG names should always be lowercase
---
Update Date: 2024-03-06

## Aggregation Statement
### COUNT
```sql
select COUNT(column1) from tableName
```
COUNT returns the number of valid rows. (not null)
Also, we can use 1 or * to represent all columns to return number of columns:
```sql
select COUNT(1) from tableName
```

### SUM
```sql
select SUM(column1) from tableName
```
SUM return sum of all values in column1. 

### MAX / MIN
```sql
select MAX(column1) from tableName
select MIN(column1) from tableName
```
MAX/MIN return max/min of all values in column1.

### AVG
```sql
select AVG(column1) from tableName
```
AVG return average of all values in column1.
If `where` is added at the end, the AVG returns conditional mean.

### ROUND
```sql
select ROUND(column1, decimalPlaces) from tableName
```
This returns rounded values. If `decimalPlaces` are omitted, it returns rounded with zero decimals.

### HAVING
```sql
select AVG(column1) from tableName
group by column2
having condition
```
A conditional statement similar to WHERE but used with aggregate functions (`COUNT()`, `MIN()`, `MAX()`, `SUM()`, `AVG()`).
The query above means that it returns averaged of column1 given different values of column2 that under condition.