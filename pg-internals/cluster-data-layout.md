# Table of contents
<!-- TOC -->
- [Table of contents](#table-of-contents)
- [Cluster](#cluster)
- [Data directory](#data-directory)
  - [Layout of PGDATA](#layout-of-pgdata)
- [System catalogs](#system-catalogs)
- [OID](#oid)
- [Another identifiers used internally](#another-identifiers-used-internally)
- [Schemas](#schemas)
- [Table spaces](#table-spaces)
- [Forks](#forks)
- [Pages](#pages)
  - [Page layout](#page-layout)
    - [Page header](#page-header)
    - [Line pointer (aka Item Id)](#line-pointer-aka-item-id)
    - [Heap tuple](#heap-tuple)
  - [TID](#tid)
- [HOT mechanism](#hot-mechanism)
- [TOAST](#toast)
- [Varlena](#varlena)
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
- **CID** is a **command ID** (entry in **WAL**);
- **TID** is a **tuple ID** (aka **Item Pointer**);

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

If schema is **not** specified explicitly in the query, the **first** appropriate schema from **search path** is chosen. Such schema is called **current schema**.<br>
The **search path** is a **list of schemas** to look in.<br>

To **show search path** there is a command: `show search_path`:
```sql
example=# show search_path;
   search_path
-----------------
 "$user", public
(1 row)
```

<br>

It is possible to **change** _search path_: `SET search_path TO 'bookings, "$user", public';`.<br>
The `$user` means a schema with the same name as the **current user**, if such schema **doesn't exist** the entry is **ignored**.<br>
Postgresql **implicitly** adds `pg_catalog` schema to the _search path_. If it is **not** named **explicitly** in the _search path_ then it is **implicitly** used **before** using _search path_.<br>

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
- **main** *fork* holds the **actual data** for a _table_ or _index_;
- **free space map** *fork* (aka **fsm** *fork*) stores information about **free space** available in a _table_ or _index_;
- **visibility map** *fork* (aka **vm** *fork*) tracks pages that **don't have** *dead tuples* and that **contains only** *frozen tuples*;
  - the **vm** *fork* is provided for *tables*, but **not** for *indexes*;
  - the *vacuum process* marks the pages in the **vm**;
- **initialization** *fork* (aka **init** *fork*);
  - the **init** *fork* is provided for **unlogged**  _tables_ and _their indexes_;
  - to create **unlogged** table use `CREATE UNLOGGED TABLE`;
  - actions with **unlogged** table are **not** written to **WAL**;

<br>

The **visibility map** is a **bitmap** that for each table page stores **two bits**:
  - the **first bit** (aka **all-visible**) is set by the *vacuum process* for pages that contain only **all-visible** tuples, in other words tuples that are accessible to **all** transactions, regardless of the snapshot used;
  - the **second bit** (aka **all-frozen**) is set by the *vacuum process* for pages that contain only **all-frozen** tuples, in other words **xmin** of such tuples can be reused as transaction id;

<br>

The *vacuum process* **skips pages** for which **all-visible** bit is **set**, because there is nothing to clean up.<br>
Besides, when a transaction tries to read a row from such a page, there is no point in checking its visibility, so an **index-only scan** can be used.<br>


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

There is built-in function `pg_stat_file('abs_path')` that returns the **size of file**.<br> 

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

The **size of segment** is **limited** and **by default** the **max size of segment** is **1 Gb**.<br>
The **size of segment can be set only during the configuration stage** when compiling PostgreSQL by parameter `--with-segsize`.<br>
When segment reaches its limit postgresql creates **new** segment with the same name and **increments** its **suffix**.<br>

<br>

# Pages
More details here: [**page layout**](https://www.postgresql.org/docs/current/storage-page-layout.html).<br>

Each _segment_ is stored as an **array of pages** (aka **blocks**) of **fixed size** and by default it is **8 Kb**.<br>
The **page size can be set only during the configuration stage** when compiling PostgreSQL by parameter `--with-blocksize`.<br>
The **pages** within each _fork_ are **numered sequentially** from **0** and these numbers are called block numbers.<br>

In a **table**, all the pages are **logically equivalent**, so a particular **item** (**row**) can be stored in any page.<br>
In an **index**, the **first** page is generally **reserved as a metapage** holding **control information**, and **there can be different types of pages within the index**, depending on the **index access method**.<br>
<br>

![page layout](/img/pg-page-layout.png)

<br>

## Page layout
| Section                       | Description                                                                                                                                                                                                         |
|:------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Page header**               | Contains **general information** about the page. **24 bytes long**. Defined by structure `PageHeaderData`.                                                                                                          |
| **Line pointers**             | **Array** of **line pointer**. A **line pointer** (aka **ItemId** or **lp**) holds **pointer to heap tuple**. **4 bytes long**. Defined by structure `ItemIdData`.                                                  |
| **Free space** (aka **hole**) | The unallocated space between **end** of the **last** _line pointer_ and the **beginning** of the **first** _item_.                                                                                                 |
| **Heap tuples**               | A **heap tuple** (aka **item** or **row** or just **tuple** or **record**) is an _individual_ **data value** itself. Every **heap tuple** has **tuple header** which is defined by structure `HeapTupleHeaderData`. |
| **Special space**             | **Index access method specific data**. Different methods store different data. **Empty** in ordinary **tables**.                                                                                                    |

<br>

**Heap tuples** are **stacked** in order **from** the **bottom** of the page.<br>
In the computer science, _this type of page_ is called a **slotted page**, and the **line pointers** correspond to a **slot array**.<br>

<br>

There are [**set of built-in functions**](https://www.postgresql.org/docs/current/pageinspect.html) to get low level info about pages.<br>
For example, get **page_header**:
```sql
SELECT * FROM page_header(get_raw_page('pg_class', 0));
```

<br>

### Page header
The **page header** is defined by structure `PageHeaderData`:
```c
typedef struct PageHeaderData
{
	PageXLogRecPtr pd_lsn;		/* LSN: next byte after last byte of xlog
								 * record for last change to this page */
	uint16		pd_checksum;	/* checksum */
	uint16		pd_flags;		/* flag bits, see below */
	LocationIndex pd_lower;		/* offset to start of free space */
	LocationIndex pd_upper;		/* offset to end of free space */
	LocationIndex pd_special;	/* offset to start of special space */
	uint16		pd_pagesize_version;
	TransactionId pd_prune_xid; /* oldest prunable XID, or zero if none */
	ItemIdData	pd_linp[FLEXIBLE_ARRAY_MEMBER]; /* line pointer array */
} PageHeaderData;
```

The **page checksums** are _enabled_ through the parameter `--data-checksums` of the `initdb` command.<br>
**By dfault** _page checksums_ are **disabled**. Checksums **cannot** be enabled on _working cluster_.<br>

There is **configuration parameter** `data_checksums`:
```sql
example=# show data_checksums;
 data_checksums
----------------
 off
(1 row)
```

<br>

### Line pointer (aka Item Id)
The **line pointer** (aka **Item Id** or **lp**) is defined by structure `ItemIdData`:
```c
typedef struct ItemIdData
{
	unsigned	lp_off:15,		/* offset to tuple (from start of page) */
				lp_flags:2,		/* state of line pointer, see below */
				lp_len:15;		/* byte length of tuple */
} ItemIdData;

/*
 * lp_flags has these possible states.  An UNUSED line pointer is available
 * for immediate re-use, the other states are not.
 */
#define LP_UNUSED		0		/* unused (should always have lp_len=0) */
#define LP_NORMAL		1		/* used (should always have lp_len>0) */
#define LP_REDIRECT		2		/* HOT redirect (should have lp_len=0) */
#define LP_DEAD			3		/* dead, may or may not have storage */
```

<br>

A **line pointer** (aka **ItemId** or **lp**) holds **pointer to heap tuple**.<br>
The **line pointers** within each _page_ are **numered sequentially** from **1**.<br>
The **line pointer number** (aka **offset number**) serves as an **index** in the **array** of line pointers.<br>

<br>

### Heap tuple
Every **heap tuple** (aka **item** or **row** or just **tuple** or **record**) has **tuple header** which is defined by structure `HeapTupleHeaderData`:
```c
typedef struct HeapTupleFields
{
	TransactionId t_xmin;		/* inserting xact ID */
	TransactionId t_xmax;		/* deleting or locking xact ID */
	union
	{
		CommandId	t_cid;		/* inserting or deleting command ID, or both */
		TransactionId t_xvac;	/* old-style VACUUM FULL xact ID */
	}			t_field3;
} HeapTupleFields;

typedef struct DatumTupleFields
{
	int32		datum_len_;		/* varlena header (do not touch directly!) */
	int32		datum_typmod;	/* -1, or identifier of a record type */
	Oid			datum_typeid;	/* composite type OID, or RECORDOID */
} DatumTupleFields;

struct HeapTupleHeaderData
{
	union
	{
		HeapTupleFields t_heap;
		DatumTupleFields t_datum;
	}			t_choice;

	ItemPointerData t_ctid;		/* current TID of this or newer tuple (or a speculative insertion token) */
	/* Fields below here must match MinimalTupleData! */
	uint16		t_infomask2;	/* number of attributes + various flags */
	uint16		t_infomask;		/* various flag bits, see below */
	uint8		t_hoff;			/* sizeof header incl. bitmap, padding */
	/* ^ - 23 bytes - ^ */
	bits8		t_bits[FLEXIBLE_ARRAY_MEMBER];	/* bitmap of NULLs */
	/* MORE DATA FOLLOWS AT END OF STRUCT */
};
```

Some fields:
- `t_xmin` holds the `txid` of the transaction that **inserted** _this tuple_;
- `t_xmax` holds the `txid` of the transaction that **deleted** or **updated** _this tuple_;
- `t_cid` holds the `cid` (**command id**), which is the **number of SQL command** that were executed **before** _this command_ was executed within the current transaction, **starting from 0**;
  - example: assume that we execute **3** `INSERT` commands within a single transaction: `BEGIN; INSERT; INSERT; INSERT; COMMIT;`;
    - if the **first** command inserts _this tuple_, `t_cid` is set to **0**
    - Ii the **second** command inserts _this tuple_, `t_cid` is set to **1**, and so on;
- `t_ctid` is used by **HOT mechanism** and holds the `tid` (**tuple identifier**) that points to `ItemId` of _this tuple_ or _new version of tuple_;

<br>

**Interpreting** the **actual data of tuple** can only be done with information obtained from other tables, mostly `pg_attribute`.<br>

<br>

## TID
A **TID** stands for **tuple ID** (aka **Item Pointer**).<br>
A **value** of **TID** is a **pair** `(page number, offset number)`, that identifies the header of **tuple**.<br>
A **TID** is also a **type of** a system column **ctid**.<br>
TIDs are used in indexes: indexes **don't** point to tuples directly, **indexes point to line pointer**.<br>
This **layer of indirection** allows to **reorder tuples inside page** during `VACUUM`: tuples are reordered on a page, but **TIDs** are **still valid** because they **point to line pointers**.<br>
The `VACUUM` deletes **dead tuple** and can reorder remaining tuples on the page to optimize space. But tuples **can only be deleted** when there are **no** active transactions that anc see them.<br> 

<br>

To obtain **TID** value for records add `ctid` to field list of `SELECT`:
```sql
example=# SELECT ctid, * FROM y;
ctid  | a
-------+---
 (0,1) | 1
(1 row)
```

<br>

# HOT mechanism
**HOT** stands for **Heap Only Tuples**. _HOT_ **avoids to update index** for cases when **indexed attribute** of row was not changed. This is reached through **HOT chains**.<br>
But HOT has restriction - the **whole HOT chain must** be presented in **one** page and **cannot** be split through pages.<br>

<br>

# TOAST
Postgresql **doesn't** allow tuples to span multiple pages and whole tuple must be resided in one page.<br>
To overcome this limitation for **large** tuples **TOAST** is used. TOAST work **transparently**: postgresql _TOASTs_ and _de-TOASTs_ values under the hood.<br>
Only **certain** data types support TOAST and are called **TOAST-able** data types.<br>
**TOAST-able** data types are such data types that can potentially have the large values.<br>
If table contains at least one attribute of TOAST-able type then postgres creates associated **TOAST table** whose **OID** is stored in the `pg_class.reltoastrelid` of **owning table**.<br>
**TOAST tables** are located in the schema `pg_toast` which is **not** in search path.<br>

<br>

**How TOAST works?**<br>
The TOAST code is triggered only when a size of row ig greater than `TOAST_TUPLE_THRESHOLD` (by default, **1/4 of one page** or **2 KB**).<br>
The TOAST code will **compress** and/or **move field values out-of-line** until the row value is shorter than `TOAST_TUPLE_TARGET` bytes (also normally 2 kB, adjustable) or no more gains can be had.<br>

<br>

**Values** of **TOAST-able types** are divided into **2 category**:
- **out-of-line**: values stored **out-of-line** in a **TOAST table**;
- **in-line**: values are stored **in-line** in **owning table**;

The **compression** technique can be used for either **in-line** or **out-of-line** data.<br>

<br>

**Out-of-line values** are divided (after compression if used) into **chunks** of at most `TOAST_MAX_CHUNK_SIZE` bytes.
**Each chunk** is stored as a **separate row** in the **TOAST table** belonging to the **owning table**.
Every TOAST table has the columns
- `chunk_id` (an OID identifying the particular **TOASTed value**);
- `chunk_seq` (a **sequence number** of the **chunk**)
- `chunk_data` (the **actual data** of the chunk);

<br>

A **unique index** on `chunk_id` and `chunk_seq` provides fast retrieval of the values.<br>

<br>

All values of **variable** data types have **varlena header**.<br>
If tuple is moved to TOAST table the owning table stores **TOAST pointer** instead tuple data. **TOAST pointer** is a part of **varlena header**.<br>

<br>

There are exist **4** different **strategies** for **storing** TOAST-able columns on disk:
- `PLAIN`: **doesn't use TOAST**, it is for columns of **non-TOAST-able** data types;
- `EXTENDED`: allows both _compression_ and _out-of-line storage_;
  - this is the **default** for most **TOAST-able** data types;
  - compression will be attempted **first**, then _out-of-line storage_ if the row is still too big;
- `EXTERNAL` allows **only** _out-of-line storage_ **without** _compression_;
  - `EXTERNAL` is good for values that have **bad** compression ratio, e.g. **JPEG** images;
  - `EXTERNAL` makes **substring operations** on long strings **faster**, because there is no need to decompress;
  - `EXTERNAL` **saves CPU time** for unuseful attempts to compress data, but `EXTERNAL` has a **penalty** of **increased** storage space;
- `MAIN` allows only _compression_ **without** _out-of-line storage_;
  - **actually**, **out-of-line** storage will still be performed for such columns, but only as a **last resort** when there is no other way to make the row small enough to fit on a page;

<br>

Each **TOAST-able** data type has **default strategy**, which can be **altered** for a given table column:
```sql
ALTER TABLE xxx ALTER COLUMN yyy SET STORAGE { PLAIN | EXTERNAL | EXTENDED | MAIN | DEFAULT };
```

<br>

The **default strategy** for each type can be figured out from the table `pg_type` by query:
```sql
SELECT DISTINCT typname, typstorage, typlen
FROM pg_type
ORDER BY typname;
                typname                 | typstorage | typlen
----------------------------------------+------------+--------
 inet                                   | m          |     -1
 int2                                   | p          |      2
 int4                                   | p          |      4
 int4range                              | x          |     -1
 int8                                   | p          |      8
 int8range                              | x          |     -1
 json                                   | x          |     -1
 jsonb                                  | x          |     -1
 macaddr                                | p          |      6
 money                                  | p          |      8
 name                                   | p          |     64
 numeric                                | m          |     -1
```

Values of **typstorage** field:
- `p` = `plain`;
- `e` = `external`;
- `m` = `main`;
- `x` = `extended`;

<br>

There are available 3 compression algorithms in postgresql:
- `pglz`: postgresql's algorithm;
- `lz4`: **faster** than `pglz`, to enable it the postgresql must be built with `–-with-lz4`;
- `zstd`;

<br>

# Varlena
All **variable length data types** have a **variable-length** (aka **varlena**) representation defined by struct `varatt_external`:
```c
typedef struct varatt_external
{
	int32		va_rawsize;		/* Original data size (includes header) */
	uint32		va_extinfo;		/* External saved size (without header) and compression method */
	Oid			va_valueid;		/* Unique ID of value within TOAST table */
	Oid			va_toastrelid;	/* RelID of TOAST table containing it */
}			varatt_external;
```

<br>

The **first 4-byte word** `va_rawsize` of any stored value contains the **total length** of the value in bytes (**including** itself).<br>
If variable data are stored **out-of-line** in a TOAST table then **varlena value** stores **TOAST pointer** instead value.<br>
TOAST uses **two bits** of the **varlena length word** (the **high-order** bits on **big-endian** machines, the **low-order** bits on **little-endian** machines), thereby limiting the logical size of any value of a TOAST-able data type to **1 GB** (230 - 1 bytes).<br>

<br>

Bit layouts for varlena headers on **little-endian** machines:
- `xxxxxx00` 4-byte length word, **aligned**, **UNcompressed** data (up to **1G**);
- `xxxxxx10` 4-byte length word, **aligned**, **compressed** data (up to **1G**);
- `00000001` 1-byte length word, **unaligned**, **TOAST pointer**;
- `xxxxxxx1` 1-byte length word, **unaligned**, **UNcompressed** data (up to **126b**);

<br>

**Explanation**:
- when **both bits are zero**, the value is an ordinary **un-TOASTed value** of the data type, and the remaining bits of the length word give the **total size** (**including** length word) in bytes;
- when the **highest-order** or **lowest-order** bit is set, the value has only a **single-byte header** instead of the normal four-byte header, and the remaining bits of that byte give the **total size** (**including** length byte) in bytes, this supports space-efficient storage of values **shorter** than **127** bytes;
- when the **highest-order** or **lowest-order** bit is set and all **remaining bits** of a **single-byte header** are **all** zero, the value is a TOAST pointer, i.e. pointer to out-of-line data;
- when the **highest-order** or **lowest-order** bit is **clear** but the **adjacent bit is set**, the content of the value has been **compressed** and must be decompressed before use;
  - compression is also possible **for out-of-line** data but the **varlena header** **does not** tell whether it has occurred — the content of the TOAST pointer tells that;
