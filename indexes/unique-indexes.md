# Unique indexes
**Indexes** are used to **enforce uniqueness** of one **column** or **tuple**.<br>
Currently, only **B-tree indexes** can be declared **unique**.<br>

```sql
CREATE UNIQUE INDEX name ON table (column [, ...]) [ NULLS [ NOT ] DISTINCT ];
```

<br>

By default, `NULL` values in a unique column are **not** considered equal, allowing **multiple** `NULL` in the column.<br>
The `NULLS NOT DISTINCT` option enforces the index to treat `NULL` as **equal**.

<br>

> **Note**:<br>
> `NULLS NOT DISTINCT` option **not** available in PostgreSQL prior **15**.<br>
> To treat `NULL` as **equal** there is **unique partial index** with `IS NULL` restriction in PostgreSQL prior **15**.

<br>

# Partial indexes
A **partial index** is an index built **over** a **subset** of a table; the **subset** is defined by a **conditional expression** (called the **predicate** of the partial index).<br>
The **index** contains entries **only** for those table **rows** that **satisfy** the **predicate**.<br>

```sql
CREATE UNIQUE INDEX [IF NOT EXISTS] name ON table (column [, ...]) WHERE predicate;
```

### Example
```sql
CREATE UNIQUE INDEX idx ON my_table (id_A, id_B) WHERE id_C IS NULL;
```

<br>


#### Use COALESCE to replace NULLs with concrete values
```sql
CREATE UNIQUE INDEX [IF NOT EXISTS] idx ON tbl USING btree (name, COALESCE(id, (0)::bigint), foo);
```