 variant of DISTINCT is DISTINCT ON. now Let’s explore this.

 
DISTINCT ON 
When distinct cannot return unique row when all columns combination is not unique then we can use distinct on clause which will give  
first row  from that set of duplicate rows.The column which we are specifying in DISTINCT ON <col_name> should also be present 
in the ORDER BY clause; otherwise you will get an error.

Example 
You can use `DISTINCT ON` to display the first of each value in `col1`:

```sql
postgres=#  SELECT DISTINCT ON (col1) col1,col2,col3 FROM test ORDER BY col1;
 col1 | col2 |    col3    
------+------+------------
    1 | abc  | 2015-09-10
    2 | xyz  | 2015-09-13
    3 | tcs   | 2015-01-15
(3 rows)
```


# DISTINCT
`DISTINCT` is used to **remove duplicate** rows from the SELECT query and only display one unique row from result set.<br>
```sql
postgres=# SELECT * FROM test;
  col1 | col2 |    col3    
------+------+------------
     1 | abc  | 2015-09-10
     1 | abc  | 2015-09-11
     2 | xyz  | 2015-09-12
     2 | xyz  | 2015-09-13
     3 | tcs  | 2015-01-15
     3 | tcs  | 2015-01-18
(6 rows)
```

1. You can use `SELECT` with `DISTINCT` to find only the non-duplicate values from column `col1`:
```sql
postgres=# SELECT DISTINCT(col1) FROM test ORDER BY col1;
  col1 
------
    1
    2
    3
(3 rows)
```

1. You can use `SELECT` with `DISTINCT` on **two** columns of the table:
```sql
postgres=#  SELECT DISTINCT col1,col2 FROM test ORDER BY 1;
 col1 | col2 
------+------
    1 | abc
    2 | xyz
    3 | tcs
(3 rows)
```
 

3. You can also use SELECT with DISTINCT on **all** columns of the table:
```sql
postgres=#  SELECT DISTINCT col1,col2,col3 FROM test ORDER BY 1;
 col1 | col2 |    col3    
------+------+------------
    1 | abc  | 2015-09-11
    1 | abc  | 2015-09-10
    2 | xyz  | 2015-09-13
    2 | xyz  | 2015-09-12
    3 | tcs  | 2015-01-15
    3 | tcs  | 2015-01-18
(6 rows)
```
