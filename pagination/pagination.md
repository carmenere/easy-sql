# Approaches
|Approach|Drawbacks|Advantages|
|:-------|:--------|:---------|
|**Limit-Offset**|1. Result **inconsistency**.<br>2. **Large offsets** are intrinsically **expensive**.|1. **Stateless**.|
|Cursor|**Statefull**. Requires to **hold** a **connection** and/or **transaction** per client.<br>Each open transaction consumes database resources, and is **not scalable** for too many clients.|1. Result **consistency**.|
|**Keyset Pagination**|1. **Lack** of **random access** (no way to jump directly to a given page without visiting prior pages to observe their maximal elements).<br>2. Requires using `ORDER BY` **over indexed columns**.|1. Result **consistency**.<br>2. Stateless|

<br>

# LIMIT (offset, limit)
Instead:
```sql
SELECT * FROM test_table ORDER BY id LIMIT 100000, 30
```

Use:
```sql
SELECT * FROM test_table JOIN (SELECT id FROM test_table ORDER BY id LIMIT 100000, 30) as b ON b.id = test_table.id
```

<br>

`LIMIT` is worked **after** rows were retrieved, so `SELECT id FROM test_table ORDER BY id LIMIT 100000, 30` retrieves rows using index, because there is **specific** indexed column.<br>
So, it is knows where row with ordinal number `100000` is.

`SELECT * FROM test_table ORDER BY id LIMIT 100000, 30` **doesn't** use index, because there are **all** columns. So, it is **doesn't** know where row with ordinal number `100000` is and return whole table.

<br>

## DECLARE CURSOR
1. **Create** cursor:
```sql
DECLARE CURSOR my_cursor
IS
SELECT * FROM customers
WHERE cust_email IS NULL
```
2. **Open** cursor:
```sql
OPEN CURSOR CustCursor
```
3. Read cursor by `FETCH`.
