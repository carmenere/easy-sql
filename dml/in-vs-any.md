# IN vs. ANY
`IN` is **equivalent** to `= ANY`, they both have the **same** execution plan.<br>

But there is an edge case:
1. **IN** with empty list **fails**:
```sql
SELECT * FROM (VALUES (1)) AS t(id) WHERE id IN ();
ERROR:  syntax error at or near ")"
```
2. **ANY** with empty list **succeeds**:
```sql
SELECT * FROM (VALUES (1)) AS t(id) WHERE id = ANY('{}');
 id
----
(0 rows)
```

<br>

`ANY` is more **versatile**, i.e., it can be combined with various operators, not just `=`.<br>
Example:
```sql
SELECT 'foo' LIKE ANY('{foo,bar}');
```
