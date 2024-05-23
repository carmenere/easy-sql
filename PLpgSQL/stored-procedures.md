# Functions vs. Stored procedures
**Stored procedures** differ from **functions** in the following ways:
- *functions* **must** **return a value** but *stored procedures* **don't**;
- it is possible **commit** and **rollback** transactions inside *stored procedures*, but **not** in *functions*;
- *stored procedure* are executed using the `CALL` statement rather than a `SELECT` statement;
- *functions* are executed using the `SELECT func(args)` statement.

<br>

# Functions
There are 2 variants to create `function`:
1. Using `CREATE FUNCTION` directly:
```sql
CREATE FUNCTION function_name(arg1 type [, ...]) RETURNS type
AS $$
DECLARE
    /* variables */
BEGIN
    /* code ... */
EXCEPTION
    /* error handling */
END;
$$ LANGUAGE plpgsql;
```
2. Using `CREATE FUNCTION` inside `DO` block and intercept exceptions:
```sql
DO $so_proc$ 
BEGIN
    CREATE FUNCTION function_name(arg1 type [, ...]) RETURNS type
    AS $$
    DECLARE
        /* variables */
    BEGIN
        /* code ... */
    EXCEPTION
        /* error handling */
    END;
    $$ LANGUAGE plpgsql;
EXCEPTION
    WHEN duplicate_object THEN NULL;
    WHEN duplicate_function THEN NULL;
END $so_proc$;
```

<br>

> **Note**:<br>
> `DECLARE` section is **optional**.<br>
> To create function that **return nothing** `void` type must be used.<br>
> To create function for **trigger** `TRIGGER` type must be used.<br>

<br>

### Access to arguments
There are 2 variants to access to passed arguments:
1. Use argument name in double quoted `"Cid"`:
```sql
CREATE OR REPLACE FUNCTION f1("arg1" bigint)
RETURNS void
AS $$
BEGIN
    ...
    WHERE t.id = "arg1";
END;
$$ LANGUAGE plpgsql;
```
2. Use `$N` notation, where `N` is an **ordinal number** of the parameter in the procedure declaration.
```sql
CREATE OR REPLACE FUNCTION f1(arg1 bigint)
RETURNS void
AS $$
BEGIN
    ...
    WHERE t.id = $1;
END;
$$ LANGUAGE plpgsql;
```

<br>

### Example
```sql
CREATE FUNCTION my_function(id bigint)
RETURNS void 
AS $$
BEGIN
    CREATE TEMP TABLE foo 
    (                                  
        id BIGINT NOT NULL, 
        name VARCHAR
    );

    INSERT INTO foo 
        SELECT baz.id, baz.name 
        FROM UNNEST(
            array[
                ('1'::bigint, 'abc'::varchar),
                ('2'::bigint, 'xyz'::varchar)
            ]
        ) as baz(id bigint, name varchar(255))
        WHERE baz.id = $1;

    DROP TABLE foo;
END;
$$ LANGUAGE plpgsql;
```

<br>

# Stored procedures
