# COALESCE
The `COALESCE(val1, val2, ... )` function returns the **first** **non-null** value in a list.<br>

```sql
SELECT COALESCE(NULL, (0)::bigint);
 coalesce
----------
        0
(1 row)
```

<br>

# NULLIF
```sql
NULLIF(argument_1, argument_2);
```

The `NULLIF` function returns a `NULL` value if `argument_1` equals to `argument_2`, otherwise, it returns `argument_1`.

<br>

Examples:
- `SELECT NULLIF (1, 1);` -- return **NULL**
- `SELECT NULLIF (1, 0);` -- return **1**

<br>

**Consider example**. The column `col_a` can contain **empty value**, then we can use `COALESCE` + `NULLIF` to convert any **empty value** to **NULL**:
```sql
SELECT
  id,
  title,
  COALESCE (
    NULLIF (col_a, ''),
    LEFT(col_b, 40)
  )
FROM
  posts;
```

<br>
