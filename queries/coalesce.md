# COALESCE
The `COALESCE(val1, val2, ... )` function returns the **first** **non-null** value in a list.<br>

```sql
SELECT COALESCE(NULL, (0)::bigint);
 coalesce
----------
        0
(1 row)
```