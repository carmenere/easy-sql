# Table of contents
<!-- TOC -->
- [Table of contents](#table-of-contents)
- [Access methods](#access-methods)
  - [The index only scan](#the-index-only-scan)
  - [Indexes with the `INCLUDE` clause](#indexes-with-the-include-clause)
  - [The bitmap index scan](#the-bitmap-index-scan)
- [Table access methods](#table-access-methods)
- [Index access method](#index-access-method)
  - [Operator class](#operator-class)
  - [Index selection](#index-selection)
<!-- TOC -->

<br>

# Access methods
An **access method** characterizes the way used to **scan** tables and **retrieve** only those rows that meet the **selection criteria**.<br>
There are 2 groups of **access methods**:
- **heap** access methods - direct access to tables whithout using indexes:
  - **sequential** scan (**seq scan**);
- **index** access methods:
  - **index** scan;
  - **index only** scan (it is **optimization** of *index scan*);
  - **bitmap index** scan (it is **optimization** of *index scan*);

<br>

The **index scans** obtains *TIDs* through reading index, and then reads tuples from pages of *table* (*heap*) using the obtained *TIDs*.<br>

<br>

For each table in the query, the **planner** looks for indexes that could optimize execution, because there can be columns for which indexes are created.<br>
The planner evaluates the **cost** of each query plan and then chooses the **cheapest** plan.<br>
For some queries **sequential scan** can be **cheaper** then **index scan**.<br>

<br>

**Cost of access** depends on both the **selectivity** of the query and the **correlation** between the **order** of tuples in the *table* and the **order** of *TIDs* in the *index*:

<br>

**Selectivity**:
- **low selectivity** of the query means query returns **many** rows, for example, when there is **no** `WHERE` clause in query or when `WHERE` clause fetches a **wide** range of rows;
- **high selectivity** of the query means query returns a **few** rows, for example, when the `WHERE` clause fetches a **narrow** range of rows;

So,
- **index scan** is good for **high selectivity**;
- **seq scan** is good for **low selectivity**;

<br>

**Correlation**:
- **high correlation**: if the **order** of tuples in the *table* has perfect correlation with the **order** of *TIDs* in the *index*, **each page will be accessed only once**: the **index scan** will sequentially go from one page to another, reading the tuples one by one;
- **low correlation**: the **index** scan process has to **jump between pages randomly** instead of reading them sequentially; in the **worst-case scenario**, the number of page accesses can reach the number of fetched tuples, in toher words: **reading happens in a random fashion**;

<br>

So, the **efficiency of an index scan is limited**: as the *correlation* **decreases**, the *number of accesses to heap pages* **rises**, and **scanning** becomes **random** rather than sequential.

For example, if the **correlation is low**, **index scanning** becomes **less attractive** for queries with **low selectivity**.<br>

<br>

## The index only scan
The *index only scan* is an **optimization** of processing *TIDs*: if an index contains all the *heap* data required by the query **extra** *table* **access** can be **avoided** and instead of TIDs, the access method can return the **actual data** associated with TIDs directly.<br>

The name suggests that *index only scan* never has to access the heap, **but it is not so**.<br>
In PostgreSQL, indexes contain **no** information on tuple **visibility** , so the access method returns **all** the heap tuples that satisfy the filter condition, **even** if the current transaction **cannot** see them. Their **visibility** is then **checked** by the indexing engine.<br>

The *index only scan* uses the **visibility map** provided for *heaps* (*tables*), in which the *vacuum process* marks the pages that contains only **all-visible** tuples (tuples that are accessible to all transactions, regardless of the snapshot used). If the *TIDs* returned by the index access method belongs to such a page, there is **no** need to check its visibility.<br>

The **cost** estimation of an **index-only scan** depends on the number of **all-visible** pages in the heap.<br>
The **index-only scan** is efficent for selecting data are **changed rarely**.<br>

<br>

## Indexes with the `INCLUDE` clause
It is not always possible to extend an index with all the columns required by a query:
• for a unique index, adding a new column **may break** the unique constraint;
• the index access method **may not provide** an operator class for the data type of the column to be added;

<br>

But it is possible to **include columns into an index without making them a part of the index key**.<br>
It will of course be impossible to perform an index scan based on the included columns.<br>

```sql
CREATE UNIQUE INDEX ON bookings(book_ref) INCLUDE (book_date);
```

<br>

## The bitmap index scan
The *bitmap index scan* is an **optimization** of processing *TIDs*: postgres fetches **all** the *TIDs* **before** accessing the *table* (*heap*) and **sort** them in ascending order based on their **page numbers**.

Unlike a regular index scan, *bitmap index scan* is represented in the **query plan by 2 nodes**:
- **bitmap index scan** gets the **bitmap** of **all** TIDs from the access method;
- the **bitmap** consists of **separate segments**, each corresponding to a single heap page;
- **bitmap heap scan** traverses the bitmap segment by segment and reads the corresponding pages **exactly once**;

<br>

Example of query plan nodes of *bitmap index scan*:
```sql
Bitmap Heap Scan on bookings (cost=54.63..7040.42 rows=2865 wid...
    Recheck Cond: (total_amount = 48500.00)
    −> Bitmap Index Scan on bookings_total_amount_idx
        (cost=0.00..53.92 rows=2865 width=0)
        Index Cond: (total_amount = 48500.00)
```

<br>

# Table access methods
A **tablea ccess method** is a **storage engines** for table.<br>
The PostgreSQL you to create various table access methods, but there is only one available out of the box at the moment - **heap**:

```sql
SELECT amname, amhandler FROM pg_am WHERE amtype = 't';
 amname |      amhandler
--------+----------------------
 heap   | heap_tableam_handler
(1 row)
```

You can specify the **engine** to use when creating a table (`CREATE TABLE ... USING`).<br>

<br>

# Index access method
The **index subsystem** consist of **index engine** which relates to **core code** of pg and **index access methods**.<br>
**Indexes** actually match a **key value** (such as the **value of the index column**) with the **TID** of tuple _containing the key value_.<br>
Each **index access method** (aka **index types**) is described by a row in the `pg_am` system catalog.<br>
The `pg_am` entry specifies a **name** and a **handler function** for the **index access method**.<br>

<br>

Postgresql provides several **built-in index access methods**:
```sql
example=# SELECT * FROM pg_am;
 oid  | amname |      amhandler       | amtype
------+--------+----------------------+--------
    2 | heap   | heap_tableam_handler | t
  403 | btree  | bthandler            | i
  405 | hash   | hashhandler          | i
  783 | gist   | gisthandler          | i
 2742 | gin    | ginhandler           | i
 4000 | spgist | spghandler           | i
 3580 | brin   | brinhandler          | i
(7 rows)

example=#
```

<br>

**Explanation**:
- **GIN** indexes are useful for **data types** that contain **multiple values** in a **single column**, for example, `arrays`, `range types`, `jsonb`.<br>
- **GiST** indexes are useful for **data types** whose values can overlap in some way. For example, hey are good for `exclude index` or `full text search`.<br>
- **SP-GiST** (**Space Partitioned GiST**) indexes are useful for **data types** whose values have **natural clustering element**, for example **country code** in telephone numbers.<br>
- **BRIN** is similar to **SP-GiST** but for **large datasets**.<br>

<br>

Postgres allows adding **new index** types by creating **extensions**.<br>

<br>

The **index access method** is responsible for:
- how data is **organized** in an index;
- how **queries** can access it;
- how to **build execution plan**;
- how operations like **insertions**, **deletions**, and **updates** are performed on the index;

<br>

Each **index access method** must implement **access method API**. The **access method API** provides the **set of routines** that define the **behavior** of a particular index type.<br>
Note, in pg, the term **routine** refers to functions or procedures that are part of an interface or API.<br>
The **access method API** is represented by the struct `IndexAmRoutine`.<br>
A structure called `IndexAMRoutine` holds pointers to the functions that **manage index operations** for that specific **index type**, i.e. which **do all of the real work**.<br>
The **index engine** just calls appropriate functions and reads appropriate properties on instances of `IndexAmRoutine`.<br>

So, each **index access method** is an instance of `IndexAMRoutine` struct.<br>

The **access method API** includes following groups of routines:
- **index creation**: function to **create** index, the _index engine_ **doesn't** know how to **create** index, it only knows which **method** to **call**;
- **index scanning**: functions to **scan** the index and **identify relevant tuples** that satisfy the query conditions;
- **insertion**, **deletion**, and **updates**: functions to **insert**, **delete** and **update** tuples;
    - for instance, in a **B-tree** index, new entries are inserted in sorted order, while a **GIN** index stores data as key-value pairs;
- **index rebuilding** and **vacuuming**: routines for rebuilding and vacuuming indexes, these processes optimize storage and ensure the index remains efficient;
- **query planning** and **index selection**;

<br>

## Operator class
The routines for an _index access method_ **don't** directly know anything about the _data types_ that the _index access method_ will operate on.<br>
To be useful, an _index access method_ must also have **one** or **more operator families** and **operator classes**.<br>
So the **index logic** is partially implemented by the **access method API** and some of the **index logic** is also implemented by the **operator class**.<br>

The **operator classes** define how _access method_ can handle a **particular data type** in **WHERE-clause operators**.<br>
The **same** _data type_ can be used by **multiple** _operator classes_.<br>
The **same** _operator class_ can be used by **multiple** _index access methods_.<br>

<br>

![mapping-am-opclass-dt](/img/access_methods2opclass.png)

<br>

An **index definition** can specify an **operator class** for each column of an index:
```sql
CREATE INDEX name ON table (column opclass [ ( opclass_options ) ] [sort options] [, ...]);
```

<br>

Consider example:
- the `GIST` is the _index access method_;
- the `inet_ops` is an **operator class**;
- for data types `cidr` and `inet` **two** `GIST`'s **operator classes** are provided:
    - the **operator class** `gist_cidr_ops`, it is the **default** opclass;
    - the **operator class** `inet_ops`;
    - for historical reasons, the `inet_ops` operator class is **not** the default class for types `inet` and `cidr`;
- to use `inet_ops` set it explicitly in **index definition**:
```sql
CREATE INDEX ON my_table USING GIST (some_column inet_ops);
```

<br>

**Operator classes** are stored in the `pg_opclass` system table. Based on the **oid** in `pg_am`, you can see the various **operators** corresponding to **operator classes** and **operator families** for each **access method**.<br>

<br>

This query shows all defined **operator classes**, their **applicable data types** and **operator family** for each **access method**:
```sql
SELECT am.amname AS index_method,
       opc.opcname AS opclass_name,
       opf.opfname AS opfamily_name,
       opc.opcintype::regtype AS indexed_type,
       opc.opcdefault AS is_default
    FROM pg_am AS am, pg_opclass AS opc, pg_opfamily AS opf
    WHERE opc.opcmethod = am.oid AND
          opc.opcfamily = opf.oid
    ORDER BY index_method, opclass_name;

index_method  |         opclass_name         |       opfamily_name       |        indexed_type         | is_default
--------------+------------------------------+---------------------------+-----------------------------+------------
 btree        | array_ops                    | array_ops                 | anyarray                    | t
 btree        | bit_ops                      | bit_ops                   | bit                         | t
 btree        | bool_ops                     | bool_ops                  | boolean                     | t
 btree        | bpchar_ops                   | bpchar_ops                | character                   | t
 btree        | bpchar_pattern_ops           | bpchar_pattern_ops        | character                   | f
 btree        | bytea_ops                    | bytea_ops                 | bytea                       | t
 btree        | char_ops                     | char_ops                  | "char"                      | t
 btree        | cidr_ops                     | network_ops               | inet                        | f
 btree        | date_ops                     | datetime_ops              | date                        | t
 btree        | enum_ops                     | enum_ops                  | anyenum                     | t
 btree        | float4_ops                   | float_ops                 | real                        | t
 btree        | float8_ops                   | float_ops                 | double precision            | t
 btree        | inet_ops                     | network_ops               | inet                        | t
 btree        | int2_ops                     | integer_ops               | smallint                    | t
 btree        | int4_ops                     | integer_ops               | integer                     | t
 btree        | int8_ops                     | integer_ops               | bigint                      | t
....
```

<br>

This query shows **all** defined operator families and **all** the operators included in **each** family:
```sql
SELECT am.amname AS index_method,
       opf.opfname AS opfamily_name,
       amop.amopopr::regoperator AS opfamily_operator
FROM pg_am am, pg_opfamily opf, pg_amop amop
WHERE opf.opfmethod = am.oid AND
  amop.amopfamily = opf.oid
ORDER BY index_method, opfamily_name, opfamily_operator;
index_method |       opfamily_name       |                      opfamily_operator
--------------+---------------------------+--------------------------------------------------------------
 btree        | integer_ops               | =(integer,bigint)
 btree        | integer_ops               | <(integer,bigint)
 btree        | integer_ops               | >(integer,bigint)
 btree        | integer_ops               | <=(integer,bigint)
 btree        | integer_ops               | >=(integer,bigint)
 btree        | integer_ops               | =(smallint,smallint)
 btree        | integer_ops               | <(smallint,smallint)
 btree        | integer_ops               | =(integer,integer)
 btree        | integer_ops               | <(integer,integer)
 btree        | integer_ops               | =(bigint,bigint)
 btree        | integer_ops               | <(bigint,bigint)
 btree        | integer_ops               | >(bigint,bigint)
 btree        | integer_ops               | <=(bigint,bigint)
 btree        | integer_ops               | >=(bigint,bigint)
 btree        | integer_ops               | =(bigint,integer)
 btree        | integer_ops               | <(bigint,integer)
 btree        | integer_ops               | >(bigint,integer)
 btree        | integer_ops               | <=(bigint,integer)
...
```

<br>

The `psql` provides commands that shows **operator classes** and **operator families**:
|Command|Description|
|:------|:----------|
|`\dAc`|List of **operator classes** for each **access method**|
|`\dAf`|List of **operator families** for each **access method**|
|`\dAo`|List of **operators** of **operator families**|

<br>

## Index selection
1. Search **attrelid**, **attname**, **attnum** and **atttypid** in **pg_attribute** by *table* and *column name*.
2. Search **oid of operator** by *operator name* and *its operand types*.
3. Search **avaliable indexes** by *table* and *column*.
4. Search **oid of operator family** (**opfamily**) in **pg_opclass** by operator class of appropriate indexes.
5. Search **operators** supported by indexes in **pg_amop** by **oid of operator** and **oid of operator class**.
6. **Select indexes** that **support appropriate operators** (found in **pg_amop**) **can be used**.

![index_selection](/img/index_selection.png)

<br>