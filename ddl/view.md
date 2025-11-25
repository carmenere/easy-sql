# Create view
```sql
CREATE VIEW name [ ( col_1 [, ...] ) ] AS query;
CREATE OR REPLACE VIEW name [ ( col_1 [, ...] ) ] AS query;
```

<br>

The `CREATE OR REPLACE VIEW` is similar, but if a view of the same name already **exists**, it is **replaced**.<br>
**Note**, when `CREATE OR REPLACE VIEW` is used we **cannot** change names of column **if** if a view of the same name already **exists**.<br>

<br>

## Recursive view
The `CREATE RECURSIVE VIEW` statement is equivalent to the following statement:
```sql
CREATE VIEW view_name
AS
  WITH RECURSIVE
    cte_name (columns) AS (
        ...
    )
  SELECT columns FROM cte_name;
```

<br>

## Materialized view
```sql
CREATE MATERIALIZED VIEW [ IF NOT EXISTS ] name
    [ (column_name [, ...] ) ]
    [ USING method ]
    [ WITH ( storage_parameter [= value] [, ... ] ) ]
    [ TABLESPACE tablespace_name ]
    AS query
    [ WITH [ NO ] DATA ]
```

<br>

The query **populates** the view at the time the command is issued.<br>
To refresh **materialized view** there is the `REFRESH MATERIALIZED VIEW name` command.<br>
The `WITH NO DATA` allows to create **empty view** and populate it later using `REFRESH MATERIALIZED VIEW name`.<br>

<br>

# Drop view
```sql
DROP VIEW [ IF EXISTS ] name [, ...] [ CASCADE | RESTRICT ];
DROP MATERIALIZED VIEW [ IF EXISTS ] name [, ...] [ CASCADE | RESTRICT ]
```

