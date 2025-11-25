# INSERT ... ON CONFLICT ...
```sql
INSERT INTO tbl(col1, col2, ...) 
VALUES (val11, val12, ... ), (val21, va22, ... )
[ ON CONFLICT [ conflict_target ] conflict_action ];
```

The `{tgt}` can be one of:
- **empty**: if `{tgt}` is **omitted** then **PK** or **any** **UNIQUE** is used;
- `(column_name)`;
- `ON CONSTRAINT constraint_name`;
- `WHERE predicate`;

The `{action}` can be one of the following:
- `DO NOTHING` means **do nothing** if the **row already exists** in the table;
- `DO UPDATE SET column_1 = value_1, ... WHERE condition` update some fields in the table if the **row already exists** in the table;

<br>

## UPSERT
**UPSERT** (**UP**date or in**SERT**) means `INSERT ... ON CONFLICT DO UPDATE SET col_1 = excluded.col_1 `:
- if row **exists** it performs `UPDATE`;
  - there is special `excluded` table is used to **reference** values **originally** proposed for insertion:
- if row **doesn't** exist it performs `INSERT`;

**Example**:
```sql
INSERT INTO account AS a (id, name, surname, address)
VALUES (1, 'foo', 'bar', 'baz')
ON CONFLICT (id)
DO UPDATE SET name=excluded.name,
    surname=excluded.surname;
```
