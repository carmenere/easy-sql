# DO blocks
**pl/pgSQL code** can be executed inside `DO` block without explicit declaration of stored procedure.<br>

There are **2** variant of syntax:
1. Simple:
```sql
DO $$
BEGIN
    /* code */
END $$;
```
2. With `DECLARE` block:
```sql
DO $$
DECLARE
    /* variables */
BEGIN
    /* code */
EXCEPTION
    /* error handling */
END $$;
```

<br>

Format of `DECLARE` section (`;` at the end of every line):
```sql
DECLARE
   counter    integer := 1;
   first_name varchar(50) := 'John';
   last_name  varchar(50) := 'Doe';
   payment    numeric(11,2) := 20.5;
```

<br>

`$$` is a **label**, **anonymous** or **named**:
- `$$` **anonymous** label;
- `$foo$` **named** label;

<br>

## LOOPS
```sql
DO $$
DECLARE
    r RECORD; 
    cnt INT; 
BEGIN
    FOR r IN 
        SELECT table_name FROM information_schema.tables 
        WHERE table_schema = 'pg_catalog' AND table_type != 'VIEW' 
        ORDER BY table_name DESC 
    LOOP 
        EXECUTE 'select count(*) cnt FROM ' || r.table_name INTO cnt; 
        RAISE NOTICE '% %', r.table_name, cnt; 
    END LOOP; 
END $$;
```

<br>

```sql
DO $$
DECLARE
    i INTEGER;
    j INTEGER;
    q TEXT;
BEGIN
    FOR i IN 1 .. 5 LOOP
        q = 'create temp table temp_table_' || i || '(';
        FOR j IN 1 .. 5 LOOP
            IF j <> 1 THEN
                q = q || ',';
            END IF;
            q = q || 'attr_' || j || ' int';
        END LOOP;
        q = q || ');';
        EXECUTE q;
    END LOOP;
END $$;
```
