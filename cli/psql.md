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
bar=# \d+ child
                                                        Table "public.child"
 Column |          Type          | Collation | Nullable |              Default              | Storage  | Stats target | Description
--------+------------------------+-----------+----------+-----------------------------------+----------+--------------+-------------
 id     | bigint                 |           | not null | nextval('child_id_seq'::regclass) | plain    |              |
 pid    | bigint                 |           | not null |                                   | plain    |              |
 name   | character varying(255) |           | not null |                                   | extended |              |
 range  | int4range              |           | not null |                                   | extended |              |
Indexes:
    "child_pkey" PRIMARY KEY, btree (id)
    "child_pid_name_key" UNIQUE CONSTRAINT, btree (pid, name) DEFERRABLE INITIALLY DEFERRED
    "child_pid_range_excl" EXCLUDE USING gist (pid WITH =, range WITH &&)
Check constraints:
    "child_forbid_empty_range" CHECK (range <> int4range(1, 1))
    "child_range_bounds" CHECK (lower(range) >= 0 AND upper(range) < 1000)
Foreign-key constraints:
    "child_pid_fkey" FOREIGN KEY (pid) REFERENCES parent(id) ON DELETE CASCADE DEFERRABLE INITIALLY DEFERRED
Access method: heap
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