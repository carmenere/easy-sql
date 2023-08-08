# psql
### Remote connection
```bash
psql postgresql://${USER_NAME}:${USER_PASSWORD}@${HOST}:${PORT}/${USER_DB}
```

<br>

### Local connection (peer auth method)
```bash
PGUSER=${USER_NAME} PGDATABASE=${USER_DB} psql
```

<br>

## psql commands
|Command|Description|
|:------|:----------|
|`\l`|List all **databases**.|
|`\c db_name`|Connect to db `db_name`.|
|`\du`|List all **users**.|
|`\dt`|List all **tables** (relations).|
|`\dt+`|List all **tables** (relations) with additional information.|
|`\d+ tbl_name`|Get detailed inforation on a table `tbl_name`.|
|`\dn`|List all **schemas**.|
|`\df`|List all **functions** and **stored procedures**.|
|`\df+`|Show code of **function** or **stored procedure**. Also `SELECT proname, prosrc FROM pg_proc WHERE proname = 'proc_name';`|
|`\dv`|List all **views**.|
|`\dT`|List all **data types**.|
|`\x`|Change query output format to **pretty-format**.|

<br>

```sql
bar=# \d my_table
                   Table "pg_temp_3.my_table"
 Column |         Type          | Collation | Nullable | Default
--------+-----------------------+-----------+----------+---------
 id     | bigint                |           | not null |
 name   | character varying(15) |           | not null |
 prefix | inet                  |           | not null |
Indexes:
    "my_table_id_prefix_excl" EXCLUDE USING gist (id WITH =, prefix inet_ops WITH &&)
```

<br>

## Set `null` placehoder
By default, `NULL` is displayed as nothing and indistinguishable from **empty string**:
```sql
bar=# select null as col;
 col
-----

(1 row)
```
<br>

`\pset null null_placeholder` set `null` placehoder to `null_placeholder`.

Example:
```sql
bar=# \pset null NULL
Null display is "NULL".

bar=# select null as col;
 col
------
 NULL
(1 row)
```

<br>

# Enable measuring query time
`\timing on`

<br>

# Get error codes
`\set VERBOSITY verbose`

<br>

**Before**:
```sql
ERROR:  relation "test_table" already exists
CONTEXT:  SQL statement "CREATE TABLE test_table(
    name varchar UNIQUE
  )"
PL/pgSQL function inline_code_block line 3 at SQL statement
```

<br>

**After**:
```sql
ERROR:  42P07: relation "test_table" already exists
CONTEXT:  SQL statement "CREATE TABLE test_table(
        name varchar UNIQUE
    )"
PL/pgSQL function inline_code_block line 3 at SQL statement
LOCATION:  heap_create_with_catalog, heap.c:1162
```