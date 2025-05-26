# psql
## Connection url
```bash
psql postgresql://${USER_NAME}:${USER_PASSWORD}@${HOST}:${PORT}/${USER_DB}
```

<br>

## Local connection (peer auth method)
```bash
PGUSER=${USER_NAME} PGDATABASE=${USER_DB} psql
```

<br>

## Parameters
### Host parameter
Cli options:
- `-h hostname`
- `--host=hostname`

Env:
- `PGHOST=hostname`

The **value** for **host parameter** can be:
- **domain name**;
- **IPv4/IPv6 address**;
- **absolute path** to file (must start with `/`);
- **comma-separated list** of allowed values:
  - each value in the list is tried in order;
  - an **empty** item selects **default behaviour**;

If **host parameter** and **hostaddr parameter** are **not specified** or **empty** the **default behaviour** is to use **UDS** in `/tmp` directory (or whatever directory was specified when PG was built).<br>
If **host parameter** contains **domain name** or **IP address**, then `libpq` will connect to server using **TCP/IP**.<br>

<br>

### Hostaddr parameter
Env:
- `PGHOSTADDR=hostaddr`

The **value** for **hostaddr parameter** is an **IPv4/IPv6 address** of host to connect to.<br>
Using **hostaddr parameter** allows the application to **avoid** a DNS lookup overhead.<br>
However, **some auth methods require host parameter**, e.g. `GSSAPI` or `SSPI` methods.<br>

<br>

### Combinations of Host and Hostaddr parameters
1. If _host parameter_ is specified **without** _hostaddr parameter_ and it is not UDS or IP addr, then DNS lookup **occurs**.
2. If _hostaddr parameter_ is specified **without** _host parameter_ the `libpq` **skips** DNS lookup. However, **some auth methods require host parameter** and in such cases the connection attempt will fail.
3. If **both** _host parameter_ and _hostaddr parameter_ are **specified** the _host parameter_ is **ignored** unless the auth method requires it in which case it will be used.
4. If **both** _host parameter_ and _hostaddr parameter_ are **not specified**, or are **empty**, the `libpq` will connect using a **UDS**.

<br>

### Port parameter
Cli options:
- `-p port`
- `--port=port`

Env:
- `PGPORT=port`

Specifies the **TCP port**. If **not set** or **empty**, the **default value** `5432` will be used.<br>

<br>

### User parameter
Cli options:
- `-U username`
- `--username=username`

Env:
- `PGUSER=username`

Specifies the username to connect as. If **not set** or **empty**, the **default value** will be used.<br>
The **default username** is the same as the **OS user** running the `psql` command.<br>

<br>


### Dbname parameter
Cli options:
- `-d dbname`
- `--dbname=dbname`

Env:
- `PGDATABASE=dbname`

Specifies the **name of db** to connect to.<br>
It can be a **connection url** `postgresql://${USER_NAME}:${USER_PASSWORD}@${HOST}:${PORT}/${USER_DB}`, if so, it must be specified as **first positional argument**.<br>
A **connection url** overrides any cli option or env.<br>

If **not set** or **empty**, the **user parameter** will be used as dbname.<br>

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
|`\dp`|List privileges.|
|`\dpp`|List default privileges.|

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

<br>

# `\gexec`
## Example 1: SELECT ... WHERE NOT EXISTS
```bash
echo "SELECT '$QUERY1', '$QUERY2' WHERE NOT EXISTS ($CHECK_QUERY)" '\gexec' | psql
```
Here `\gexec` will execute `'$QUERY1'`, then `'$QUERY2'` if `$CHECK_QUERY` returns **nothing**.<br>

<br>

## Example 2: SELECT ... WHERE EXISTS
```bash
echo "SELECT '$QUERY1', '$QUERY2' WHERE EXISTS ($CHECK_QUERY)" '\gexec'
```
Here `\gexec` will execute `'$QUERY1'`, then `'$QUERY2'` if `$CHECK_QUERY` returns **at least one row**.