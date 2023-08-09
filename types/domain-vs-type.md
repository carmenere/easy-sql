# `CREATE DOMAIN` vs `CREATE TYPE`
- `CREATE DOMAIN` creates a user-defined **data type** with **additional constraints** such as `NOT NULL`, `CHECK`, etc.
- `CREATE TYPE` creates a user-defined **composite data type** and it can be used in *stored procedures* as the data type of *returning value*.

<br>

# Enum
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