# Table of contents
<!-- TOC -->
* [Table of contents](#table-of-contents)
* [Cluster](#cluster)
* [Data directory](#data-directory)
  * [Layout of PGDATA](#layout-of-pgdata)
* [System catalogs](#system-catalogs)
* [OID](#oid)
* [Another identifiers used internally](#another-identifiers-used-internally)
* [Schemas](#schemas)
* [Table spaces](#table-spaces)
* [Forks](#forks)
<!-- TOC -->

<br>

# Cluster
A **database cluster** is a **collection** of **databases** (aka **catalogs**) that are managed by a **single server instance**.<br>
So, in postgresql the term _database cluster_ **doesn't** mean a group of DBMS servers.<br>

The word **cluster** is defined by the **SQL standard**, e.g., in **SQL-92** specification:
- a **cluster** is an implementation-defined **collection of catalogs**;
- exactly **one** cluster is associated with an **SQL session**.

<br>

**One instance** of postgresql server runs on a **single listening post** and manages a **single database cluster**.<br>
**Multiple clusters**, managed by **different** server instances, **can exist on the same host**.<br>

<br>

Most of DB provides this full hierarchy: `Cluster > Catalog > Schema > Table`.

So,
- **single server instance** is a **cluster**.
- *cluster* has **catalogs**. (**catalog** = **database**)
- *catalogs* have **schemas**. (**schema** is a **namespace** for tables)
- *schemas* have **tables**.
- *tables* have **rows**, **columns** and **cells** (**values**).

<br>

**Fully qualified name** of **table** inside cluster: `catalog.schema.table`. 

<br>

# Data directory
The **cluster's data directory** (aka **data directory** or **data area** or **datadir**) is a **directory** where postgresql stores **all subdirectories and files** that are referred to a _database cluster_.<br>
There is `initdb` utility to initialize a **new** _database cluster_: it **creates** _data directory_ and **populates** it with some initial data.<br>
The **path** to **data directory** must be specified through `-D` **option** or `PGDATA` **environment variable**, thee is **no** default.<br>
That's why **data directory** is also called **PGDATA**.<br>

<br>

During initialization `initdb` creates **3 databases**:
- **template0**;
- **template1** - **template for any database** that will be created in this cluster;
- **postgres** - **default database**, that can be used at will;

<br>

To obtain a path to **PGDATA** there is a command: `show data_directory`:
```text
example=# show data_directory;
data_directory
---------------------------------
/opt/homebrew/var/postgresql@17
(1 row)
```

<br>

## Layout of PGDATA
The full list of main files and subdirectories are listed here [**layout of PGDATA**](https://www.postgresql.org/docs/current/storage-file-layout.html).<br>
The main files and subdirectories:
|Item|Description|
|:---|:----------|
|`PG_VERSION`|A file containing the **major version number** of PostgreSQL|
|`postgresql.conf`|A file containing various **configuration parameters**|
|`pg_hba.conf`|A file containing **auth policies**|
|`base`|Subdirectory containing **per-database subdirectories**|
|`global`|Subdirectory containing **cluster-wide tables**, e.g. tables that are **shared between all databases**|
|`pg_tblspc`|Subdirectory containing **symbolic links to tablespaces**|
|`pg_wal`|Subdirectory containing **WAL** (Write Ahead Log) **files**|
|`pg_xact`|Subdirectory containing **transaction commit status data**|
|`pg_stat`|Subdirectory containing **files for the statistics subsystem**|

<br>

A **database** is a **subdirectory** under the `base` directory in a **PGDATA**. The **name of database** is identical to its **OID**.<br>
To obtain an **OID** of database by its name run:
```sql
example=# SELECT oid FROM pg_database WHERE datname = 'example';
oid
--------
 101424
(1 row)
```

So, all tables corresponding to `example` database are located in `${PGDATA}/base/101424`.<br>

<br>

# System catalogs
**System catalogs** (aka **system tables**) are **regular tables** where RDBMS stores **meta information** about all objects in cluster:
- **schema metadata**: information about tables and columns;
- **built-in entities**: _datatypes_, _functions_, _operators_, etc;

<br>

Most **system catalogs** are copied from `template1` database during database creation and become **database specific**.<rb>
But there are a few **system catalogs** that are **shared across all databases** in a cluster.<br>
The `psql` client tool provides a **set of special commands** to deal with _system catalogs_.<br>

<br>

Conventions:
- names of **all tables** of _system catalogs_ have prefix `pg_`, e.g. `pg_database`;
- names of **all columns** begin with **3 letter code**, which is usually corresponds first 3 letter of name of table after prefix `pg_`
  - for table `pg_database` all its columns will begin with `dat`;
  - table `pg_class` was previously called `pg_relation`, but its columns still have prefix `rel`;
- the **primary key column** in all tables has name `oid`;

<br>

# OID
**OID** stands for **Object IDentifier**.<br>
The **OIDs** are used as **primary keys** of type `oid` in various tables from system catalogs. the **OID** is implemented as an **unsigned 4-byte integer**.<br>
**All** tables in system catalogs share the **same space** of **OID**, in other words, **all OID** columns in different tables share the **same sequence**.<br>

<br>

To obtain an **OID** of table by its name run:
```sql
example=# SELECT oid FROM pg_class WHERE relname = 'foo';
oid
--------
 101427
(1 row)
```

<br>

**Tables** and **indexes** are **internally managed** by **OIDs**, while their **segments** are managed by **relfilenode** attribute.<br>
Note, that value of **relfilenode** attribute **doesn't** always **match** their **OID**.<br>
Avoid assuming that table or index OID and their relfilenode are the same. Some operations, like `ALTER TABLE`, `TRUNCATE`, `REINDEX` can **change name of segments** while **preserving** the **OID** of _table_ or _index_.<br>

<br>

Get attributes for table with name `foo`:
```sql
SELECT * FROM pg_attribute WHERE attrelid = 'foo'::regclass;
```

The expression `'foo'::regclass` is converted to **right** OID value even if there are **multiple tables** named `foo` in different schemas.<br>
The converter for each alias type **handles** the **table lookup** according to the **search path**, so it does right things automatically.<br>
Without such conversion under the hood you would have to write subqueries that look like:
```sql
SELECT * FROM pg_attribute WHERE attrelid = (SELECT oid FROM pg_class WHERE relname = 'foo');
```

<br>

There are also several **OID alias types** (aka **OID types**), each named **reg**_something_, each alias type can be casted to OID.<br>
Mapping between **OID alias types** and **system entities** is here [**OID types**](https://www.postgresql.org/docs/current/datatype-oid.html).<br>

| OID type       | Entity       | Description    |
|:---------------|:-------------|:---------------|
| `oid`          | any          | numeric OID    |
| `regclass`     | pg_class     | relation name  |
| `regnamespace` | pg_namespace | namespace name |
| `regrole`      | pg_authid    | role name      |
| `regtype`      | pg_type      | data type name |

<br>

# Another identifiers used internally
- **XID** is a **transaction ID** (also abbreviated **xact**);
- **CID** is a **command ID** (in WAL);
- **TID** is a **tuple** (aka **row** or **record**) ID;
  - **TID** is a type of a system column **ctid**;
  - a **value** of **TID** is a pair `(block number, item id number)`, that identifies the physical location of row within its table;

<br>

# Schemas
**Schemas** are **namespaces** for objects in database. Any object in database must belong to some schema.<br>
Schemas allow to have multiple tables with the same name.<br>

There are some predefined schemas for any database:
- `public` - **default schema**;
- `pg_catalog` - contains **system catalogs**;
- `pg_toast` - contains tables for **TOAST objects**;
- `pg_temp` - contains **temporary tables**;
- `information_schema` - alternate representation of **system catalogs** as **SQL standard** requires;

<br>

**Qualified names** consist of the **schema** name and **table** name, separated by a dot: `schema.table`.<br>
**Unqualified names** consist of just **table** name. Qualified names are **tedious** to write and tables are often referred to by unqualified names.<br>

If schema is **not** specified explicitly in the query, the **first** appropriate schema from **search path** is chosen.<br>
The **search path** is a **list of schemas** to look in.<br>

To **show current search path** there is a command: `show search_path`:
```sql
example=# show search_path;
   search_path
-----------------
 "$user", public
(1 row)
```

<br>

The `$user` means a schema with the same name as the current user, if such schema doesn't exist the entry is ignored.<br>
The **first schema** in the _search path_ that **exists** is used for **created objects**.<br>
Postgresql **implicitly** adds `pg_catalog` schema to the _search path_. If it is **not** named **explicitly** in the _search path_ then it is **implicitly** used **before** using _search path_.<br>
It is possible to **change** _search path_: `SET search_path TO myschema,public;`.<br>

<br>

# Table spaces
**Table space** is a **directory** in file system. The **same** _table space_ can be used by **different** _databases_ and the **same** _database_ can use **different** _table spaces_.<br>
During cluster initialization 2 table spaces are created:
- `pg_default` is located in `PGDATA/base` and is used as **default table space**;
- `pg_global` is located in `PGDATA/global` and stores all objects **common** for **entire** cluster;

<br>

To create **table space** run:
```sql
CREATE TABLESPACE tablespace_name LOCATION 'path_to_directory';
```
The directory `path_to_directory`
- **must exist** (`CREATE TABLESPACE` will **not** create it);
- **should be empty**;
- **must be owned** by the PostgreSQL system user;
- must be specified by an **absolute path**;

<br>

Example:
1. By default `pg_tblspc` subdirectory is **empty**:
```shell
ls -hal /opt/homebrew/var/postgresql@17/pg_tblspc
total 0
drwx------   2 an.romanov  admin    64B Jul 15 18:37 .
drwx------  26 an.romanov  admin   832B Jul 15 08:46 ..
```
2. Run query:
```sql
CREATE TABLESPACE bar LOCATION '/Users/an.romanov/Downloads/space_bar';
```
3. List `PGDATA/pg_tblspc` again:
```shell
ls -hal /opt/homebrew/var/postgresql@17/pg_tblspc
total 0
drwx------   3 an.romanov  admin    96B Jul 15 18:42 .
drwx------  26 an.romanov  admin   832B Jul 15 08:46 ..
lrwx------   1 an.romanov  admin    37B Jul 15 18:42 101432 -> /Users/an.romanov/Downloads/space_bar
```

The directory `PGDATA/pg_tblspc/101432` is a just **symlink** to a **table space** `/Users/an.romanov/Downloads/space_bar`.<br>
The **empty** directory `PG_17_202406281` will be created in the **table space**.<br>

The **table space** is alternate `base` directory and any **database** or **table** that is created in **specific table space** will be located in **its** directory.<br>

<br>

Example:
1. Create database in specific table space `bar`: `CREATE DATABASE fizzbaz WITH TABLESPACE = 'bar';`;
2. The **directory of database** is located in `PG_17_202406281`:
```shell
ls -hal /Users/an.romanov/Downloads/space_bar/PG_17_202406281
total 0
drwx------    3 an.romanov  staff    96B Jul 15 18:55 .
drwx------@   4 an.romanov  staff   128B Jul 15 18:51 ..
drwx------  300 an.romanov  staff   9.4K Jul 15 18:55 101434
```

<br>

# Forks
Every _table_ and _index_ is represented with **several type of forks**.<br>
Each **type of fork** holds a **specific type of data** for a _table_ or _index_:
- **main** fork holds the **actual data** for a _table_ or _index_;
- **free space map** (aka **fsm**) fork stores information about **free space** available in a _table_ or _index_;
- (for _tables only_) **visibility map** (aka **vm**) fork tracks pages that don't have dead tuples;
- (for **unlogged**  _tables_ and _their indexes_) **initialization** (aka **init**) fork;
  - to create **unlogged** table use `CREATE UNLOGGED TABLE`;
  - actions with **unlogged** table are **not** written to **WAL**;

<br>

The **naming convention** for **forks** is as follows:
- each _table_ and _index_ has a **relfilenode** attribute is a _system catalogs_ ant this attribute is of type **oid**;
- name of **main fork** is `relfilenode` as is;
- name of **fsm fork** is `relfilenode` plus suffix `_fsm`;
  - **fsm** forks are created only when needed;
  - to force creation of **fsm** fork call `VACUUM` for table;
- name of **vm fork** is `relfilenode` plus suffix `_vm`;
- name of **init fork** is `relfilenode` plus suffix `_init`;

<br>

All **forks** are stored in _files_ called **segments**.<br>
In other words, a **fork** is a **set of segments** all of which have the same name.<br>

<br>

The **naming convention** for **segments** is as follows:
- the **first segment** of each fork **doesn't** have **sequential number**;
- **subsequent segments** of each fork are named: `%forkname%.1`, `%forkname%.2`, etc;

<br>

There is function `pg_relation_filepath('table')` that returns the **relative** from `PGDATA` **path** for particular relation:
```sql
example=# SELECT pg_relation_filepath('y');
pg_relation_filepath
----------------------
 base/101424/101438
(1 row)
```

<br>

There is function `pg_stat_file('abs_path')` that returns the **size of file**.<br> 

Knowing `base/101424/101438` we can obtain **size of file** `base/101424/101438`:
```sql
example=# SELECT size FROM pg_stat_file('/opt/homebrew/var/postgresql@17/base/101424/101438');
size
------
 8192
(1 row)
```

<br>

**Segments** are stored in the directory: `PGDATA/base/%db_OID%`, where `%db_OID%` is an **OID of database** to which segments are belonged.<br>

The **size of segment** is **limited** and **by default** the **max size of segment** is **1 Gb**. This limit **can be changed only at compile time** by parameter `--with-segsize`.<br>
When segment reaches its limit postgresql creates **new** segment with the same name and **increments** its **suffix**.<br>
