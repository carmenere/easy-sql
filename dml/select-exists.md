# SELECT EXISTS
`SELECT EXISTS` returns exactly one value: `true` or `false`.<br>

```sql
SELECT EXISTS (SELECT * FROM tbl WHERE id > 0);
 exists 
--------
 t
(1 row)

SELECT EXISTS (SELECT * FROM tbl WHERE id < 0);
 exists 
--------
 f
(1 row)
```

<br>

# SELECT TRUE
`SELECT EXISTS` returns **zero** or **more** values.<br>

```sql
SELECT true FROM tbl WHERE id < 0;
 bool 
------
(0 rows)


SELECT true FROM tbl WHERE id > 0;
 bool 
------
 t
 t
 t
 t
 t
```
