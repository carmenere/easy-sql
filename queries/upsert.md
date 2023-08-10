# INSERT ... ON CONFLICT ...
```sql
INSERT INTO tbl(col1, col2, ...) 
VALUES (val11, val12, ... ), (val21, va22, ... )
ON CONFLICT {tgt} {action};
```

<br>

The `{tgt}` can be one of:
- `(column_name)`;
- `ON CONSTRAINT constraint_name`;
- `WHERE predicate`;

<br>

The `{action}` can be one of the following:
- `DO NOTHING` means **do nothing** if the **row already exists** in the table;
- `DO UPDATE SET column_1 = value_1, ... WHERE condition` update some fields in the table if the **row already exists** in the table;

<br>

## UPSERT
`UPSERT` logic:
- if row **exists** perform `UPDATE`;
- if row **doesn't** exist perform `INSERT`;

<br>

In other words, `UPSERT` means `INSERT ... ON CONFLICT UPDATE ... `.

<br>

## EXCLUDED
The `SET` and `WHERE` clauses in `ON CONFLICT DO UPDATE` have access
- to the **existing row** using the **table's name** (or an **alias**);
- to **rows** proposed for insertion using the special `excluded` table;

### Example
```sql
INSERT INTO account AS a (id, name, surname, address)
VALUES (1, 'foo', 'bar', 'baz')
ON CONFLICT (id)
DO UPDATE SET
    name=EXCLUDED.name,
    surname=EXCLUDED.surname;
```
