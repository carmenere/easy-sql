# DELETE
```sql
DELETE FROM foo WHERE id = 1;
```

<br>

# DELETE .. USING ..
```sql
WITH t AS (SELECT id FROM unnest(ARRAY[1,2,3]) AS t(id))
DELETE FROM foo
USING t WHERE foo.id = t.id;
```
