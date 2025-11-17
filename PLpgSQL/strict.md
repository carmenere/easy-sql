# STRICT
`STRICT` is an optional modifier in PostgreSQL's `CREATE FUNCTION` statement.<br>
It is a **shorthand** for the longer phrase `RETURNS NULL ON NULL INPUT`.<br>
When a function is declared `STRICT`, PostgreSQL first checks every argument passed at runtime. If **at least one** argument is `NULL`, the function body is **not** executed and the **result is immediately** `NULL`.<br>
In other words, `STRICT` **prevents** the function body from running if **at least one argument** is `NULL`.<br>
If you supply both `STRICT` and `RETURNS NULL ON NULL INPUT`, PostgreSQL treats them as the same.<br>
`STRICT` is **incompatible** with the `CALLED ON NULL INPUT` attribute (**the default**) and will cause a syntax error if both appear.<br>

<br>

## Use cases with SQL STRICT
- propagate NULLs without extra code inside functions;
- guarantee that **expensive functions are not executed** with incomplete data;
- enforce business rules that disallow partial input;

<br>

SQL `STRICT` syntax
```sql
CREATE OR REPLACE FUNCTION function_name (arg1 data_type, ...)
RETURNS return_type
LANGUAGE plpgsql
STRICT  -- or: RETURNS NULL ON NULL INPUT
AS $$
BEGIN
    -- function body only runs when all args are NOT NULL
END;
$$;
```
