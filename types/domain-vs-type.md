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

<br>

## enum_range
`enum_range` **returns all values** of the input **enum** type in an **ordered** **array**.<br>

```sql
CREATE TYPE rainbow AS ENUM ('red', 'orange', 'yellow', 'green', 'blue', 'purple');
CREATE TYPE
```

<br>

```sql
select enum_range(null::rainbow);
              enum_range
---------------------------------------
 {red,orange,yellow,green,blue,purple}
(1 row)
```
