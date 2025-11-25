# Table of contents
<!-- TOC -->
- [Table of contents](#table-of-contents)
- [Constraints](#constraints)
  - [`DEFAULT`, `NOT NULL`, `UNIQUE` and `CHECK`](#default-not-null-unique-and-check)
    - [Per column](#per-column)
    - [Per table](#per-table)
  - [`PRIMARY KEY`](#primary-key)
    - [Per column](#per-column-1)
    - [Per table](#per-table-1)
  - [`FOREIGN KEY`](#foreign-key)
    - [Per column](#per-column-2)
    - [Per table](#per-table-2)
    - [Cascade operations](#cascade-operations)
- [List all constraints](#list-all-constraints)
- [Deferrable constraint](#deferrable-constraint)
  - [Set deferrable constraint in table](#set-deferrable-constraint-in-table)
  - [Alter deferrable constraint in table](#alter-deferrable-constraint-in-table)
- [Exclude constraint](#exclude-constraint)
- [Errors](#errors)
  - [PostgreSQL error codes](#postgresql-error-codes)
  - [Class 23 — Integrity constraint violation](#class-23--integrity-constraint-violation)
<!-- TOC -->

<br>

# Constraints
- only **per column**:
  - `DEFAULT expression`;
  - `NOT NULL`
    - this constraint **forbids** to have `NULL` values in column
    - this constraint is **functionally equivalent** to `CHECK (column_name IS NOT NULL)`;
- **per column** or **per table**:
  - `UNIQUE`
  - `CHECK`
  - `PRIMARY KEY`
    - automatically creates B-tree index;
    - there can only be **one** `PRIMARY KEY` in a table;
  - `FOREIGN KEY`
  - `EXCLUDE`

<br>

## `DEFAULT`, `NOT NULL`, `UNIQUE` and `CHECK`
### Per column
```sql
CREATE TABLE foo1 (
    name TEXT UNIQUE,
    value1 INTEGER NOT NULL DEFAULT 222,
    value2 INTEGER NOT NULL DEFAULT 500 CHECK (value1 > 100 AND value2 < 1000)
);
```

<br>

### Per table
```sql
CREATE TABLE foo2 (
    name TEXT,
    author TEXT,
    value1 INTEGER NOT NULL DEFAULT 222,
    value2 INTEGER NOT NULL DEFAULT 500,
    UNIQUE (name, author),
    CHECK (value1 > 100 AND value2 < 1000)
);
```

```sql
CREATE TABLE foo3 (
    name TEXT,
    author TEXT,
    value1 INTEGER NOT NULL DEFAULT 222,
    value2 INTEGER NOT NULL DEFAULT 500,
    CONSTRAINT unique_name_author UNIQUE (name, author),
    CONSTRAINT check_val1_val2 CHECK (value1 > 100 AND value2 < 1000)
);
```

<br>

## `PRIMARY KEY`
### Per column
```sql
CREATE TABLE parent_table1 (
    id BIGSERIAL PRIMARY KEY
);
```

<br>

### Per table
```sql
CREATE TABLE parent_table2 (
    id BIGSERIAL,
    name TEXT,
    PRIMARY KEY (id, name)
);
```

```sql
CREATE TABLE parent_table3 (
    id BIGSERIAL,
    name TEXT,
    CONSTRAINT pk_id_name PRIMARY KEY (id, name)
);
```

<br>

## `FOREIGN KEY`
### Per column
```sql
CREATE TABLE child_table1 (
    pid BIGINT REFERENCES parent_table1 (id),
    value TEXT
);
```

<br>

If name of **PK** was **omitted** in `REFERENCES tbl (name_of_pk)`, then will be used name of **PK** in the **referenced table**, `id` in the example below:
```sql
CREATE TABLE child_table1 (
    pid BIGINT REFERENCES parent_table1,
    value TEXT
);
```

<br>

### Per table
```sql
CREATE TABLE child_table2 (
    pid BIGINT,
    pname TEXT,
    value TEXT,
    FOREIGN KEY (pid, pname) REFERENCES parent_table2 (id, name)
);
```

```sql
CREATE TABLE child_table3 (
    pid BIGSERIAL,
    pname TEXT,
    CONSTRAINT fk_id_name FOREIGN KEY (pid, pname) REFERENCES parent_table3 (id, name)
);
```

<br>

### Cascade operations
**Cascade operations** refer to the **actions triggered automatically** on the related **child** table(s) when a **change** occurs in a **parent** table.<br>

*Cascade operations*:
- `ON DELETE CASCADE | NO ACTION | RESTRICT | SET NULL | SET DEFAULT` which is invoked when a **referenced column** is **deleted**;
  - `NO ACTION` (**by default**) means that the deletion in the **referenced** table is allowed to proceed, but the **FK** constraint is still required to be **satisfied**, so this operation may fail if **FK** constraint was not **satisfied** after deletion. It **can be deferred**;
  - `CASCADE` specifies that when a **referenced** row is **deleted**, row(s) **referencing** it should be automatically **deleted** as well;
  - `RESTRICT` **prevents deletion** of a referenced row and it **cannot be deferred**;
  - `SET NULL` when the **referenced** row is **deleted** sets the **referencing** column(s) in the **referencing** row(s) to be set to `NULL`;
  - `SET DEFAULT` if an action specifies `SET DEFAULT` but the **default value** would **not** satisfy the foreign key constraint, the operation will **fail**;
  - the actions `SET NULL (col_1, col_2, ... )` and `SET DEFAULT (col_1, col_2, ... )` **can take a list of columns** to specify which columns to set;
- `ON UPDATE CASCADE | NO ACTION | RESTRICT | SET NULL | SET DEFAULT` which is invoked when a **referenced column** is **changed** (**updated**);
  - `CASCADE` means that the **updated** values of the **referenced** column(s) should be **copied** into the **referencing** row(s);
  - `NO ACTION` **allow the update** to proceed and the FK constraint will be checked against the state after the update;
  - `RESTRICT` **will prevent the update** to run even if the state after the update would still satisfy the constraint;
  - the actions `SET NULL` and `SET DEFAULT` **cannot** take a list of columns;

<br>

```sql
CREATE TABLE child_table4 (
    pid BIGSERIAL,
    pname TEXT,
    CONSTRAINT fk_id_name FOREIGN KEY (pid, pname)
        REFERENCES parent_table3 (id, name)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
```

<br>

# List all constraints
```sql
SELECT
    tc.table_name, tc.constraint_name, constraint_type, kcu.column_name,
    ccu.table_name AS foreign_table_name,
    ccu.column_name AS foreign_column_name
FROM 
    information_schema.table_constraints AS tc
JOIN 
    information_schema.key_column_usage AS kcu
    ON tc.constraint_name = kcu.constraint_name
JOIN 
    information_schema.constraint_column_usage AS ccu
    ON ccu.constraint_name = tc.constraint_name;
```

<br>

# Deferrable constraint
## Set deferrable constraint in table
```sql
...
FOREIGN KEY (pid) REFERENCES parent(id) ON DELETE CASCADE DEFERRABLE INITIALLY DEFERRED,
UNIQUE (pid, name) DEFERRABLE INITIALLY DEFERRED,
...
```

<br>

## Alter deferrable constraint in table
```sql
ALTER TABLE tbl1 ALTER CONSTRAINT fk1 DEFERRABLE INITIALLY DEFERRED;
```

<br>

# Exclude constraint
1. Activate **extension** `btree_gist`:
```sql
CREATE EXTENSION btree_gist;
CREATE EXTENSION
```
2. Use **exclude** constraint `EXCLUDE USING ... `:
```sql
CREATE TABLE bookings (
    vehicle_id SERIAL PRIMARY KEY,
    travel_dates tstzrange NOT NULL,
    EXCLUDE USING gist (vehicle_id WITH =, travel_dates WITH &&)
);
```

<br>

```sql
CREATE TABLE my_table
(                                  
    id BIGINT NOT NULL,
    name VARCHAR(15) NOT NULL,
    prefix inet NOT NULL,
    EXCLUDE USING gist (id WITH =, prefix inet_ops WITH &&)
);
```
Here `inet_ops`	is an **operator class**.

<br>

# Errors
## PostgreSQL error codes
**Error codes** are specified in the **SQL-92 standard**.
[PostgreSQL Error Codes](https://www.postgresql.org/docs/current/errcodes-appendix.html)

<br>

## Class 23 — Integrity constraint violation
|Error Code|Condition Name|
|:---------|:-------------|
|23000|**integrity_constraint_violation**|
|23001|**restrict_violation**|
|23502|**not_null_violation**|
|23503|**foreign_key_violation**|
|23505|**unique_violation**|
|23514|**check_violation**|
|23P01|**exclusion_violation**|

<br>

> **Note**:<br>
> **Not-null constraint** hasn't **name** and field "**constraint**" in **error message** from pg is filled by `NONE`.

<br>

```sh
23514   check_violation:
            table: customers
            column: NONE
            data_type: NONE
            constraint: customers_name_check
            code: 23514
            message: new row for relation "customers" violates check constraint "customers_name_check"
            detail: Failing row contains (43091324936, ).
            hint: NONE
            where: NONE
            schema: public
```

<br>

```sh
23502   not_null_violation:
            table: customers
            column: id
            data_type: NONE
            constraint: NONE
            code: 23502
            message: null value in column "id" violates not-null constraint
            detail: Failing row contains (null, null).
            hint: NONE
            where: NONE
            schema: public
```


