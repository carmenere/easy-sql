# Table of contents
<!-- TOC -->
- [Table of contents](#table-of-contents)
- [Data type](#data-type)
- [Datatypes in PostgreSQL](#datatypes-in-postgresql)
  - [Boolean type](#boolean-type)
  - [Numeric types](#numeric-types)
    - [Integer types](#integer-types)
    - [Serial types](#serial-types)
    - [Floating-point types](#floating-point-types)
    - [Arbitrary precision numbers](#arbitrary-precision-numbers)
  - [Character types](#character-types)
  - [Date/Time types](#datetime-types)
  - [Arrays](#arrays)
  - [Network address types](#network-address-types)
  - [JSON types](#json-types)
  - [Geometric types](#geometric-types)
  - [Text search types](#text-search-types)
  - [Binary data types](#binary-data-types)
  - [Bit string types](#bit-string-types)
  - [Monetary types](#monetary-types)
  - [UUID type](#uuid-type)
  - [XML type](#xml-type)
  - [Enums](#enums)
  - [`CREATE DOMAIN` vs `CREATE TYPE` (Enum)](#create-domain-vs-create-type-enum)
<!-- TOC -->


<br>

# Data type
A **data type** (or simply **type**) is usually specified by
- a set of **allowed** values;
- a set of **allowed operations** on these values;
- a set of **constraints** on these values;
- a **representation** of these values as machine types (**memory layout**).

<br>

# Datatypes in PostgreSQL
1. **Boolean type**
2. **Numeric types**
3. **Character types**
4. **Date/Time types**
5. **Network address types**
6. **JSON types**
7. **Geometric types**
8.  **Text search types**
9.  **Binary data types**
10. **Bit string types**
11. **Monetary types**
12. **UUID type**
13. **XML type**
14. **Range types**
15. **Arrays**
16. **Composite types**
17. **Enums**
18. **Domain types**

<br>

## Boolean type
|Type|Aliase|Description|
|:---|:-----|:----------|
|`boolean`|`bool`|logical Boolean (**true**/**false**)|

<br>

**Strings**, that can be casted to `true` value:
- `'true'::bool`;
- `'t'::bool`;
- `'yes'::bool`;
- `'y'::bool`;
- `'on'::bool`;
- `'1'::bool`;

<br>

**Strings**, that can be casted to `false` value:
- `'false'::bool`;
- `'f'::bool`;
- `'no'::bool`;
- `'n'::bool`;
- `'off'::bool`;
- `'0'::bool`;

<br>

Is possible to omit comparison with boolean value in `WHERE` clauses and write only a name of column: instead `WHERE col1 = 'yes'::bool` write `WHERE col1`.<br>

<br>

## Numeric types
### Integer types 
|Type|Aliase|Description|
|:---|:-----|:----------|
|`smallint`|`int2`|signed **2-byte** integer|
|`integer`|`int`, `int4`|signed **4-byte** integer|
|`bigint`|`int8`|signed **8-byte** integer|

<br>

### Serial types
|Type|Aliase|Description|
|:---|:-----|:----------|
|`smallserial`|`serial2`|**autoincrementing** `smallint`|
|`serial`|`serial4`|**autoincrementing** `integer`|
|`bigserial`|`serial8`|**autoincrementing** `bigint`|

<br>

**Serial type** reprsents a whole **group of command**:
```sql
CREATE SEQUENCE foo_seq;
CREATE TABLE bar (col1 INTEGER NOT NULL DEFAULT nextval('foo_seq'));
ALTER SEQUENCE foo_seq OWNED BY bar.col1;
```

<br>

**Example**<br>

```sql
CREATE TABLE tbl1 (
    col1 SERIAL
);
```

is **equal** to:

```sql
CREATE SEQUENCE tbl1_foo1_seq;
CREATE TABLE tbl1 (
    col1 integer NOT NULL DEFAULT nextval('tbl1_foo1_seq')
);
ALTER SEQUENCE tbl1_foo1_seq OWNED BY tbl1.col1;
```

<br>

```sql
bar=# \d+ tbl1
                                                 Table "public.tbl1"
 Column |  Type   | Collation | Nullable |              Default               | Storage | Stats target | Description
--------+---------+-----------+----------+------------------------------------+---------+--------------+-------------
 col1   | integer |           | not null | nextval('tbl1_foo1_seq'::regclass) | plain   |              |
Access method: heap
```

<br>

### Floating-point types
|Type|Aliase|Description|
|:---|:-----|:----------|
|`real`|`float4`|**single precision** floating-point number (**4 bytes**), implemets of *IEEE 754*|
|`double precision`|`float`, `float8`|**double precision** floating-point number (**8 bytes**), implemets of *IEEE 754*|

<br>

### Arbitrary precision numbers
|Type|Aliase|Description|
|:---|:-----|:----------|
|`numeric [ (precision, scale) ]`|`decimal [ (precision, scale) ]`|**arbitrary precision** /prɪˈsɪʒ.ən/ numbers|

<br>

**Valid syntax** for `numeric [ (precision, scale) ]`:
- `numeric (precision, scale)`
  - the **max** `precision` value that can be explicitly specified is **1000**
- `numeric (precision)`, means `scale = 0`
  - the **max** `precision` value that can be explicitly specified is **1000**
- `numeric` **without any** `precision` or `scale` creates an **unconstrained numeric**, limits for an **unconstrained numeric**:
  - up to **131072** digits **before** the *decimal point*;
  - up to **16383** digits **after** the *decimal point*;

where:
- `precision` is the **total count** of digits in the **whole number**; 
- `scale` is the **count** of digits in the **fractional part**, to the **right** of the *decimal point*;

<br>

If the **scale** of a value is being inserted **greater** than the **declared scale** of the column in the table, the system will **round** the value to the specified number of fractional digits. Then, if the number of digits in the resulting number **exceeds** the **declared precision**, an **error** is raised.<br>

<br>

**Example**: the value is being inserted **cannot** be rounded:
```sql
SELECT '999.999'::numeric(5,2);
ERROR:  numeric field overflow
DETAIL:  A field with precision 5, scale 2 must round to an absolute value less than 10^3.
```

<br>

**Example**: the value is being inserted has the **same scale** or **can** be rounded:
```sql
SELECT '999.99'::numeric(5,2);
 numeric
---------
  999.99
(1 row)

SELECT '999.994'::numeric(5,2);
 numeric
---------
  999.99
(1 row)
```

<br>

Special values:
- `NaN` **not a number**, the `NaN` **greater** any other **normal value** of `numeric` type; 

<br>

## Character types
|Type|Aliase|Description|
|:---|:-----|:----------|
|`character [ (n) ]`|`char [ (n) ]`|**fixed-length** character string|
|`character varying [ (n) ]`|`varchar [ (n) ]`|**variable-length** character string|
|`text`| |**variable-length** character string|

<br>

Variants to escape quote:
- **double** `''`:
```sql
SELECT 'Ann''s' AS possessive_case;
 possessive_case
-----------------
 Ann's
(1 row)
```
- **double** `$$`
```sql
SELECT $$Ann's$$ AS possessive_case;
 possessive_case
-----------------
 Ann's
(1 row)
```
- **C style escapes**:
```sql
SELECT E'Ann\'s' AS possessive_case;
 possessive_case
-----------------
 Ann's
(1 row)
```

<br>

## Date/Time types
|Type|Aliase|Description|
|:---|:-----|:----------|
|`date`| |calendar date (year, month, day)|
|`time [ (p) ] [ without time zone ]`| |time of day (**no** time zone)|
|`time [ (p) ] with time zone`|`timetz`|time of day, **including** time zone|
|`timestamp [ (p) ] [ without time zone ]`| |date and time (**no** time zone)|
|`timestamp [ (p) ] with time zone`|`timestamptz`|date and time, **including** time zone|
|`interval [ fields ] [ (p) ]`| |time span|

<br>

Date and time format is defined with `datestyle` configuration parameter, by default it has following value:

```sql
SHOW datestyle;
 DateStyle
-----------
 ISO, MDY
(1 row)
```

- the **left** `ISO` defines format for **output** values;
- the **right** `MDY` defines format for **input** values **only**;
  - `SELECT '22-09-2025'::date;` returns **error**, because the **right** is set to **MDY**;
  - `SELECT '09-22-2025'::date;;` returns `2025-09-22`, because the **right** is set to **MDY** and the **left** is set to **ISO**;
  - the order **YMD**, e.g. `'2025-09-22'::date`, is a **universal** and is **always acceptable** and **doesn't** depend on the **right** value of `datestyle`;
  - but if the **first** is set to `Postgres`, then the **right** defines format for **input** and **output** values;

<br>

To change `datestyle`:
```sql
SET datestyle TO 'ISO, DMY';

SHOW datestyle;
 DateStyle
-----------
 ISO, DMY
(1 row)

SET datestyle TO default;
```

<br>

The functions `current_date`, `current_time` and `current_timestamp`:
```sql
SELECT current_date;
 current_date
--------------
 2025-11-24
(1 row)

SELECT current_time;
    current_time
--------------------
 16:39:56.864914+00
(1 row)

SELECT current_timestamp;
       current_timestamp
-------------------------------
 2025-11-24 16:40:36.031135+00
(1 row)

```

<br>

The format of `interval` type: `quantity unit [quantity unit ...] [ago]`.
Examples:
- `1 year 2 months`;
- `1 year 2 months ago`, `ago` means pg adds minus to all values;

<br>

```sql
SELECT ('2025-09-30'::timestamp - '2025-01-30'::timestamp)::interval;
 interval
----------
 243 days
(1 row)

SELECT '2025-09-30'::timestamp + '100 days'::interval;
      ?column?
---------------------
 2026-01-08 00:00:00
(1 row)
```

<br>

## Arrays
To specify an **array** add `[]` after type name, e.g. `'{1,2,3}'::integer[]` or `'{"abc", "xyz"}'::text[]`.<br>

**Notations**:
- `{item1, item2, ... }::some_type[]`
```sql
SELECT '{1,2,3}'::integer[];
  int4
---------
 {1,2,3}
(1 row)

SELECT '{"abc", "xyz"}'::text[];
   text
-----------
 {abc,xyz}
(1 row)
```
- `ARRAY [1, 2]`
```sql
SELECT ARRAY [1, 2];
 array
-------
 {1,2}
(1 row)
```
- **slice**: `column[x:y]`
```sql
UPDATE .. SET column[x:y] = ARRAY[3,4];
```

<br>

**Functions**:
- `array_append(column, value)`
- `array_prepend(value, column)`
- `array_remove(column, value)` finds value `value` and removes it
- `array_position(column, value)` finds the **first** value `value` and returns its **index**;
- `unnest(array1, ...)` **converts** arrays to columns;
- `array_cat(arr1, arr2)` **concatenates** 2 arrays;

<br>

Examples:
```sql
SELECT array_append(ARRAY[1,2], 55);
 array_append
--------------
 {1,2,55}
(1 row)

SELECT array_prepend(55, ARRAY[1,2]);
 array_prepend
---------------
 {55,1,2}
(1 row)

SELECT array_position(ARRAY[5,77], 77);
 array_position
----------------
              2
(1 row)

SELECT array_remove(ARRAY[5,77], 77);
 array_remove
--------------
 {5}
(1 row)

SELECT * FROM unnest(ARRAY['a', 'b', 'c'], ARRAY[1,2,3]) AS foo(col1, col2);
 col1 | col2
------+------
 a    |    1
 b    |    2
 c    |    3
(3 rows)
```

<br>

**Operators**:
- `arr1 @> arr2` return **true** if `arr1` contains all elements of `arr2`;
- `arr1 && arr2` return **true** if intersection of `arr1` and `arr2` is **not** empty;
- `arr || val` concatenation

<br>

```sql
SELECT '{1,2,3}'::integer[] || 55;
  ?column?
------------
 {1,2,3,55}
(1 row)

SELECT '{1,2,3}'::integer[] || '{55,77}'::integer[];
   ?column?
---------------
 {1,2,3,55,77}
(1 row)
```

<br>

## Network address types
|Type|Aliase|Description|
|:---|:-----|:----------|
|`cidr`| |IPv4 or IPv6 **network address**|
|`inet`| |IPv4 or IPv6 **host address**|
|`macaddr`| |**MAC address**|
|`macaddr8`| |**MAC address** (**EUI-64 format**)|

<br>

## JSON types
|Type|Description|
|:---|:----------|
|`json`|**textual** JSON data: **insert** is **faster**, but **select** is **slower**|
|`jsonb`|**binary** JSON data, decomposed: **insert** is **slower**, but **select** is **faster**|

<br>

Operators:
- `obj ? key_name` returns **true** if oblect `obj` **contains** key `key_name`;
- `obj->key_name` operation `->` means **access** to field of object `obj`;
- `obj1 @> obj2` returns **true** if `obj1` contains **all** elements of `obj2`;
- `obj1 || '{"abc": 22}'::jsonb` **adds** a new **key** and its **value** to the object `obj1`;
- `obj1 - 'abc'` **removes** the **key** from the object `obj1`;

<br>

## Geometric types
|Type|Description|
|:---|:----------|
|`box`|rectangular box on a plane|
|`circle`|circle on a plane|
|`line`|infinite line on a plane|
|`lseg`|line segment on a plane|
|`path`|geometric path on a plane|
|`point`|geometric point on a plane|
|`polygon`|closed geometric path on a plane|

<br>

## Text search types
|Type|Description|
|:---|:----------|
|`tsquery`|text search query|
|`tsvector`|text search document|

<br>

## Binary data types
|Type|Description|
|:---|:----------|
|`bytea`|binary data (aka **byte** **a**rray)|

<br>

## Bit string types
|Type|Aliase|Description|
|:---|:-----|:----------|
|`bit [ (n) ]`| |**fixed-length** bit string|
|`bit varying [ (n) ]`|`varbit [ (n) ]`|**variable-length** bit string|

<br>

## Monetary types
|Type|Description|
|:---|:----------|
|`money`|currency amount|

<br>

## UUID type
|Type|Description|
|:---|:----------|
|`uuid`|universally unique identifier|

<br>

## XML type
|Type|Description|
|:---|:----------|
|`xml`|universally unique identifier|

<br>

## Enums
```sql
CREATE TYPE rainbow AS ENUM ('red', 'orange', 'yellow', 'green', 'blue', 'purple');

select enum_range(null::rainbow);
              enum_range
---------------------------------------
 {red,orange,yellow,green,blue,purple}
(1 row)
```

<br>

Create `enum` **if not exists**:
```sql
DO $$ BEGIN
    CREATE TYPE colors AS ENUM (
        'black', 
        'white'
    );
EXCEPTION
    WHEN duplicate_object THEN NULL;
END $$;
```

<br>

## `CREATE DOMAIN` vs `CREATE TYPE` (Enum)
- `CREATE DOMAIN` creates a user-defined **data type** with **additional constraints** such as `NOT NULL`, `CHECK`, etc.
- `CREATE TYPE` creates a user-defined **composite data type** and it can be used in *stored procedures* as the data type of *returning value*.

<br>