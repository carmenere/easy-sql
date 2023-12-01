# DISTINCT
Both `DISTINCT col_a, col_b` and `DISTINCT ON (col1, col2) col1, col2, col3` **remove all duplicate rows**:
- `DISTINCT col_a, col_b` select all rows `col_a, col_b` with **non-duplicate** tuples `(col_a, col_b)`;
- `DISTINCT ON (col1, col2) col1, col2, col3` select all rows `col1, col2, col3` with **non-duplicate** tuples `(col1, col2)`;

<br>

> **Note**:<br>
> Use the `ORDER BY` clause with the `DISTINCT ON` or `DISTINCT` to make the result set **predictable**.

<br>

## Examples
Consider table:
```sql
SELECT * FROM test;
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

<br>

1. Display only the **non-duplicate** values from column `col1`:
```sql
SELECT DISTINCT(col1) FROM test ORDER BY col1;
  col1 
------
    1
    2
    3
(3 rows)
```

<br>

2. Display only rows with **non-duplicate** values from column `col1`:
```sql
SELECT DISTINCT ON (col1) col1,col2,col3 FROM test ORDER BY col1;
 col1 | col2 |    col3
------+------+------------
    1 | abc  | 2015-09-10
    2 | xyz  | 2015-09-13
    3 | tcs  | 2015-01-15
(3 rows)
```

<br>

3. Display only rows with **non-duplicate** tuples `(col1, col2)`:
```sql
 SELECT DISTINCT col1,col2 FROM test ORDER BY 1;
 col1 | col2 
------+------
    1 | abc
    2 | xyz
    3 | tcs
(3 rows)
```

<br>

4. You can also use `SELECT` with `DISTINCT` on **all** columns of the table:
```sql
 SELECT DISTINCT col1,col2,col3 FROM test ORDER BY 1;
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
