# Error handling in PGSQL
## PostgreSQL error codes
**Error codes** are specified in the **SQL-92 standard**.
[PostgreSQL Error Codes](https://www.postgresql.org/docs/current/errcodes-appendix.html)

<br>

## Custom exception
For **custom exception** you can use only **SQLSTATE** (aka **error code**) values in range `[7xxxx; 9xxxx]` and range `[Txxxx, Zxxxx]`.<br>

There is special syntax to set **SQLSTATE**:
```sql
RAISE 'Duplicate user ID: %', user_id USING ERRCODE = '70001';
```

<br>

## `EXCEPTION` section
There is special section `EXCEPTION` to handle errors:
```sql
EXCEPTION
    WHEN condition_name1 THEN
        /* Here is the code that handles this error */
    WHEN condition_name2 THEN
        /* Here is the code that handles thos error */
    ...
```

<br>

There is special **condition name** `OTHERS` which means **any other error**.

<br>

### Example
1. Create some table:
```sql
DO $$
BEGIN
    CREATE TABLE test_table(
        name varchar UNIQUE
    );
EXCEPTION
    WHEN duplicate_table THEN
        RAISE NOTICE 'MSG: %, ERRCODE: %', SQLERRM, SQLSTATE;
        RAISE;
END $$;
```

<br>

2. Insert some values into table:
```sql
DO $$
BEGIN
    INSERT INTO test_table(name) VALUES('abc');
    INSERT INTO test_table(name) VALUES('abc');
EXCEPTION
    WHEN unique_violation THEN
        RAISE NOTICE 'MSG: %, ERRCODE: %', SQLERRM, SQLSTATE;
        RAISE;
END $$;
```

<br>

3. Rename constraint
```sql
DO $$ BEGIN
    ALTER TABLE child RENAME CONSTRAINT child_forbid_empty_range TO forbid_empty_range;
EXCEPTION
    WHEN undefined_object THEN NULL;
END $$;
```
Here, the error `undefined_object` occurs when constraint with name `child_forbid_empty_range` doesn't exist.

<br>

4. Create `ENUM`:
```sql
DO $$ BEGIN
    CREATE TYPE colors AS ENUM (
        'black', 
        'white'
    );
EXCEPTION
    WHEN duplicate_object THEN NULL;
END $$;
```

<br>
 
The `SQLSTATE` variable contains **error code**.<br>
The `SQLERRM` function returns the **error message** associated with an **error code**.<br>
The second `RAISE;` **reraises original** exception.<br>

