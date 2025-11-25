# Table of contents
<!-- TOC -->
- [Table of contents](#table-of-contents)
- [List all indexes in db](#list-all-indexes-in-db)
- [Indexes](#indexes)
  - [Create index](#create-index)
  - [Drop index](#drop-index)
  - [Multicolumn index](#multicolumn-index)
  - [Unique index](#unique-index)
  - [Partial index](#partial-index)
    - [Use COALESCE to replace NULLs with concrete values](#use-coalesce-to-replace-nulls-with-concrete-values)
  - [Include indexes](#include-indexes)
<!-- TOC -->

<br>

# List all indexes in db
```sql
SELECT
    tablename,
    indexname,
    indexdef
FROM
    pg_indexes
WHERE
    schemaname = 'public'
ORDER BY
    tablename,
    indexname;
```

<br>

# Indexes
Tuples in relations are stored in **unordered manner**.<br>
**Index** is a separate structure which store **reverse mapping** from values to tuples where values are and all values are **sorted**.<br>

<br>

## Create index
```sql
CREATE INDEX name ON table (col_1);
```

<br>

## Drop index
```sql
DROP INDEX name;
```

<br>

## Multicolumn index
```sql
CREATE INDEX name ON table (col_1 , col_2, ...);
```

<br>

## Unique index
**Indexes** are used to **enforce uniqueness** of one column or more columns.<br>

```sql
CREATE UNIQUE INDEX name ON table (column [, ...]) [ NULLS [ NOT ] DISTINCT ];
```

<br>

By default, `NULL` values in a unique column are **not equal**, allowing **multiple** `NULL` in the column.<br>
The `NULLS NOT DISTINCT` option **enforces** the index to treat `NULL` as **equal**.<br>

<br>

> **Note**:<br>
> `NULLS NOT DISTINCT` option **not** available in PostgreSQL prior **15**.<br>
> To treat `NULL` as **equal** there is **unique partial index** with `IS NULL` restriction in PostgreSQL prior **15**.

<br>

## Partial index
A **partial index** is an index built **over** a **subset** of a table; the **subset** is defined by a **predicate of the index**.<br>
A **predicate of the index** is a conditional expression that selects rows to be indexed.<br>
The **index** contains entries **only** for those table **rows** that **satisfy** the **predicate**.<br>
The *predicate* can contain columns that **differ** from **indexed columns**.<br>

<br>

**General form**:
```sql
CREATE UNIQUE INDEX [IF NOT EXISTS] name ON table (column [, ...]) WHERE predicate;
```

<br>

**Example**:
```sql
CREATE UNIQUE INDEX idx ON my_table (id_A, id_B) WHERE id_C IS NULL;
```

Planner chooses **partial index** if condition in `WHERE` is **equal** or **can be reduced** to a **predicate of the index**.<br>

<br>

**Example**:
```sql
CREATE INDEX idx_foo ON foo (id) WHERE xx > 100;
```

For this `SELECT` the planner **will use** index `idx_foo`:
```sql
SELECT FROM foo WHERE xx > 110;
```

For this `SELECT` the planner will **not** use index `idx_foo`:
```sql
SELECT FROM foo WHERE xx > 80;
```

<br>

### Use COALESCE to replace NULLs with concrete values
```sql
CREATE UNIQUE INDEX [IF NOT EXISTS] idx ON tbl USING btree (name, COALESCE(id, (0)::bigint), foo);
```

<br>

## Include indexes
It is not always possible to extend an index with all the columns required by a query:
- for a **unique index**, adding a new column may break the unique constraint;
- the **index access method** may not provide an operator class for the data type of the column to be added;

But it is **possible** to **include columns into an index without making them a part of the index key**. It will of course be impossible to perform an index scan based on the included
columns.

```sql
CREATE UNIQUE INDEX ON bookings(book_ref) INCLUDE (book_date);
```

<br>