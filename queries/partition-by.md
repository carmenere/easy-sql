# PARTITION BY
```sql
SELECT * FROM (
    SELECT *, count(*)
    OVER
    (PARTITION BY
        name,
        status,
        comment
    ) AS count
    FROM tbl
) AS mytbl
WHERE mytbl.count > 1;
```

<br>

# Select first row from every group
```sql
WITH summary AS (
    SELECT  id, 
            tstz, 
            operation, 
            ROW_NUMBER() OVER(PARTITION BY id ORDER BY tstz ASC) AS rk
    FROM logs)
SELECT s.*
  FROM summary s
 WHERE s.rk = 1;



SELECT s.* FROM (
    SELECT  id, 
            tstz, 
            operation, 
            ROW_NUMBER() OVER(PARTITION BY id ORDER BY tstz ASC) AS rk
    FROM logs
) AS s
WHERE s.rk = 1;
```