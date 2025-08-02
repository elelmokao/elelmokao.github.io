---
title: SQL learning 1 - Manipulation
date: 2024-02-15 00:00:00 +0800
categories: [CheatSheet, SQL]
tags: [sql, learning]
# TAG names should always be lowercase
---
Update Date: 2024-02-15

## Manipulation Statements
### Create a new table
```sql
create table table_name (
    column1 datatype,
    column2 datatype,
);
```

### Modify columns
```sql
alter table table_name
-- add a new col
add column_name datatype;
-- remove a col
drop column column_name;
-- rename column
rename column old_column_name to new_column_name;
```
To change data type of a column, there are 3 syntaxes for different SQL database engine: 
```sql
alter table table_name
-- For SQL Server / MS Access
alter column column_name datatype;
-- For My SQL / Oracle (< version 10G)
modify column column_name datatype;
-- For Oracle (version 10 G and later)
modify column_name datatype;
```


### Insert new row(s)
```sql
-- insert into columns in order
insert into table_name
values (value1, value2);

-- insert into columns by name
insert into table_name (column1, column2)
values (value1, value2);
```

### Delete rows
```sql
delete from table_name
where some_column = some_value;
```
If the WHERE clause is omitted, all records will be deleted.

### Update rows
```sql
update table_name
set columns1 = value1, column2 = value2
where some_column = some_value;
```