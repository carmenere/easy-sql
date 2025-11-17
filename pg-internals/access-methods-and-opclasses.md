# Table of contents
<!-- TOC -->
- [Table of contents](#table-of-contents)
- [Index access method](#index-access-method)
  - [Operator class](#operator-class)
<!-- TOC -->

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

For each table in the query, the **planner** looks for indexes that could optimize execution, because there can be columns for which indexes are created.<br>
The **planner** then **evaluates the cost** of using each available index to access the data.<br>

**Types of scanning**:
- **sequential scans** if **no** appropriate index is found;
- **bitmap scans** if **multiple indexes** might be used together for filtering;
- **index scans** if a **single index** provides the best cost;

In index scans postgresql obtains TIDs through reading index, and then reads tuples using the obtained TIDs.<br>

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
