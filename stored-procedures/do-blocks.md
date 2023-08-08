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
