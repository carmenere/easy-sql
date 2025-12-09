# Table of contents
<!-- TOC -->
- [Table of contents](#table-of-contents)
- [GROUP BY](#group-by)
  - [GROUPING SETS, ROLLUP, CUBE](#grouping-sets-rollup-cube)
- [HAVING](#having)
- [Aggregate functions](#aggregate-functions)
- [Window functions](#window-functions)
  - [`OVER` clause](#over-clause)
  - [Window frame](#window-frame)
  - [Defaults](#defaults)
  - [Types of window functions](#types-of-window-functions)
  - [Examples](#examples)
    - [Example](#example)
- [Examples](#examples-1)
  - [Case: get first row from every group](#case-get-first-row-from-every-group)
    - [CTE version](#cte-version)
    - [Subquery version](#subquery-version)
  - [Case: get top N](#case-get-top-n)
    - [Preparation](#preparation)
    - [Select TOP 2 src\_ip per TOP 5 uri](#select-top-2-src_ip-per-top-5-uri)
      - [CTE](#cte)
      - [Subqueries](#subqueries)
  - [Case: get sales per country + city](#case-get-sales-per-country--city)
    - [Preparation](#preparation-1)
    - [Select](#select)
<!-- TOC -->

<br>

# GROUP BY
The `GROUP BY` clause **divides** the rows returned from the `SELECT` statement **into groups** by colums.<br>

<br>

General syntax of the `GROUP BY` clause:
```sql
SELECT 
   column_1, 
   column_2,
   aggregate_function(column_3)
FROM 
   table_name

GROUP BY 
   column_1,
   column_2;
```

<br>

The following example uses *multiple columns* in the `GROUP BY` clause:
```sql
SELECT 
	customer_id, 
	staff_id, 
	SUM(amount) AS amount
FROM 
	payment

GROUP BY 
	staff_id, 
	customer_id

ORDER BY 
    customer_id;
```

<br>

## GROUPING SETS, ROLLUP, CUBE
It is possible to perform multiple `GROUP BY` in one query without `UNION`:
- `GROUP BY GROUPING SETS(c.country_id, p.city_id);`
- `GROUP BY CUBE(p.payment_type_id, c.country_id, p.city_id);`
- `GROUP BY ROLLUP(p.payment_type_id, c.country_id, p.city_id);`

<br>

# HAVING
The `HAVING` clause is processed **after** the `GROUP BY` clause, so you **cannot** refer to the **aggregate function** specified in the SELECT list by using the column alias.<br>

```sql
The following query will fail:
SELECT
    column_name1,
    column_name2,
    aggregate_function (column_name3) AS aliase
FROM
    table_name
GROUP BY
    column_name1,
    column_name2
HAVING
    aliase > value;
```

<br>

Instead, you must use the **aggregate function** expression in the `HAVING` clause explicitly as follows:
```sql
SELECT
    column_name1,
    column_name2,
    aggregate_function (column_name3) alias
FROM
    table_name
GROUP BY
    column_name1,
    column_name2
HAVING
    aggregate_function (column_name3) > value;
```

<br>

# Aggregate functions
**Aggregate functions** perform calculations on a **set of rows** (**groups of data**) and return a **single summary value**.<br>
*Aggregate functions* are combined with the `GROUP BY` clause to perform calculations on specific groups of data.<br>

All aggregate functions **ignore** `NULL` values.<br>

For **each group**, you can apply an **aggregate function**: 
- `sum(col)` calculates the **sum of values** in column `col` **per** the group;
- `count(col)` calculates the **number of items** in column `col` **per** the group, **excludes** `NULL` values;
- `count(*)` **includes** `NULL` values;
- `min(col)` finds **max value** in column `col` **per** the group;
- `max(col)` finds **min value** in column `col` **per** the group;
- `avg(col)` calculates the **average of values** in column `col` **per** the group;

<br>

# Window functions
## `OVER` clause
Every *function* `func(col_1)` followed by `OVER ( PARTITION BY ... ORDER BY ... )` clause becomes **window function**.<br>
Unlike *aggregate functions* that **collapse rows** into a single result, **window functions** return a value **for each row** in the result set.<br>

There 2 variant of syntax:
- `window_definition` is **directly** specified inside `OVER` clause:
```sql
SELECT <func_name>(<column>) [ FILTER ( WHERE filter_clause ) ] OVER ( window_definition ) AS t
```
- the `OVER` clause **references** to *existing name of window*:
*window* is **directly** defined in list of columns:
```sql
SELECT <func_name>(<column>) [ FILTER ( WHERE filter_clause ) ] OVER window_name AS t
FROM foo
WINDOW window_name AS ( window_definition )
```

where **window_definition** has the syntax:
```sql
[PARTITION BY <partition_columns>]
[ORDER BY <sort_columns>]
[ frame_clause ]
```

<br>

**Examples**:
- `window_definition` is **directly** specified inside `OVER` clause:
```sql
SELECT func_name(col_1) OVER ( PARTITION BY col_2 ORDER BY col_3 DESC NULLS FIRST ) AS t
ORDER BY col_7;
```
- the `OVER` clause **references** to *existing name of window*:
```sql
SELECT
    func_name(col_1) OVER win AS t1,
    func_name(col_2) OVER win AS t2,
    func_name(col_3) OVER win AS t3
FROM foo
WINDOW win AS ( PARTITION BY col_4 ORDER BY col_5 )
ORDER BY col_9;
```

<br>

The `OVER()`clause is used to define the **window frame** (aka **frame**, **window**).<br>
A **window frame** is a **group of rows** that will be passed to the **window function**. The **window function** is **applied** to **window frame**.<br>
**Empty** `OVER ()` means that **window frame** is the **whole result**.<br>

<br>

The `OVER` clause has 3 **optional** subclauses to **customize** the **window**:
- `PARTITION BY <partition_columns>`
  - **divides** the *resulting rows* into **non-overlapping** subsets of rows (aka **partitions**);
  - each **partition** contains only rows with the **same** values in **all** columns, specified in `PARTITION BY`;
  - **window functions** are applied **separately** to each partition, as if each were a **separate** data set;
  - if you **omit** the `PARTITION BY` clause, the **window function** will treat the **whole result** as a **single partition**;
- `ORDER BY <sort_columns>`
  - **sorts** rows in **each** partition **independently**;
  - the `ORDER BY` clause uses the `NULLS FIRST` or `NULLS LAST` option to specify whether **nullable** values should be **first** or **last** in the result set;
  - the **default** is `NULLS LAST` option;
- `frame_clause`
  - it defines how many rows to include **before** and **after** the **current row** in the **window frame**;

<br>

**Example**<br>
![win_funcs_partition](/img/win_funcs_partition.png)

In the above image the `PARTITION BY country` divides all rows of *result set* into 2 **partitions** where all rows inside partiton have the **same value** in column `country`:
- all rows with `country = RUS` form **partition 1**;
- all rows with `country = ITA` form **partition 2**;

<br>

## Window frame
A **window frame** is a **subset** of rows in the **current partition** that are somehow **related** to the **current row**.<br>
A *window frame* is always bound to **current row**.<br>
The *window frame* is evaluated **separately** within each partition.<br>
In other words, a *window frame* refines which rows within that partition are included in the calculation.<br>

A **peer group** contains all **consecutive rows** (aka **peer rows**) with the exact same value in **all** columns specified in `ORDER BY` clause (aka **sorting columns**).

<br>

A **frame clause** (aka **window frame definition**) syntax:
```sql
mode BETWEEN frame_start AND frame_end [ frame_exclusion ]
```
- the **mode** can be one of:
  - `ROWS`
  - `RANGE`
  - `GROUPS`
- the **frame_start** and **frame_end** can be one of:
  - `UNBOUNDED PRECEDING`: means that the frame **starts** with the **first row** of the partition;
  - `offset PRECEDING`: the *behaviour* depends on **mode**;
  - `CURRENT ROW`: the *behaviour* depends on **mode**;
  - `offset FOLLOWING`: the *behaviour* depends on **mode**;
  - `UNBOUNDED FOLLOWING`: means that the frame **ends** with the **last row** of the partition;
- the **frame_exclusion** can be one of:
  - `EXCLUDE CURRENT ROW`: *excludes* the current row from the frame;
  - `EXCLUDE GROUP`: *excludes* the **current row** and **all** its **peers** from the frame;
  - `EXCLUDE TIES`: *excludes* **any peers** of the current row from the frame, but **not** the current row **itself**;
  - `EXCLUDE NO OTHERS`: simply specifies explicitly the **default behavior** of **not** excluding the current row or its peers;

<br>

**Restrictions**:
- `frame_start` **cannot be** `UNBOUNDED FOLLOWING`;
- `frame_end` **cannot be** `UNBOUNDED PRECEDING`;
- `frame_end` **cannot** appear **earlier** than `frame_start`;
  - for example: `RANGE BETWEEN CURRENT ROW AND offset PRECEDING` is **not** allowed;

<br>

**Shorter versions**:
- it is possible to use **shorter versions** for *window frames definitions*:
- `mode BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` is the same as `mode UNBOUNDED PRECEDING`;
- `mode BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING` is the same as `mode UNBOUNDED FOLLOWING`;
- `mode BETWEEN offset PRECEDING AND CURRENT ROW` is the same as `mode offset PRECEDING`;
- `mode BETWEEN CURRENT ROW AND offset FOLLOWING` is the same as `mode offset FOLLOWING`;
- `mode BETWEEN CURRENT ROW AND CURRENT ROW` is the same as `mode CURRENT ROW`;

<br>

The behaviour of `offset PRECEDING`, `offset FOLLOWING` and `CURRENT ROW` **depends on** *frame mode*:
- in `ROWS` mode:
  - - the *window frame* is **precise** and it **cannot** change *from row to row*;
  - the `CURRENT ROW` is an **exactly one row**;
  - `offset` must be **non-null**, **non-negative integer** `n` and it means **exactly** `n` *rows* **before** or **after** the *current row*;
    - for example, `1 PRECEDING` means **exactly one** row **before** the **current** one;
- in `RANGE` mode:
  - the *window frame* is **dynamic** and it **can** change *from row to row*;
  - the `CURRENT ROW` is **not** just a *current row*, instead it is a **peer group** (*current row* and **all** its *peer rows*);
  - a **peer group** is **included entirely** in the *frame*;
  - these options require that the `ORDER BY` clause specify **exactly one column**;
  - the `offset` specifies the **maximum difference** between the value in the column specified in `ORDER BY` of the *current row* and all rows **before** or **after** the *current row*;
    - the `n PRECEDING` includes **group of all rows** where values are in the **range** `[value_in_current_row - n, value_in_current_row]`;
    - the `n FOLLOWING` includes **group of all rows** where values are in the **range** `[value_in_current_row, value_in_current_row + n]`;
- in `GROUPS` mode:
  - the *window frame* is **dynamic** and it **can** change *from row to row*;
  - the `CURRENT ROW` is **not** just a *current row*, instead it is a **peer group** (*current row* and **all** its *peer rows*);
  - a **peer group** is **included entirely** in the *frame*;
  - `offset` must be **non-null**, **non-negative integer** `n` and it means **exactly** `n` **different** *groups of rows with the sme values* **before** or **after** the *current group*;
    - `n PRECEDING` includes `n` **groups of rows** with the same values in the column specified in`ORDER BY` **before** *current group*;
    - `n FOLLOWING` includes `n` **groups of rows** with the same values in the column specified in`ORDER BY` **after** *current group*;

<br>

The `ROWS` mode:<br>
![win_funcs_rows_range_groups](/img/win_funcs_rows_range_groups_rows.png)

<br>

The `RANGE` mode:<br>
![win_funcs_rows_range_groups](/img/win_funcs_rows_range_groups_range.png)

<br>

The `GROUPS` mode:<br>
![win_funcs_rows_range_groups](/img/win_funcs_rows_range_groups_groups.png)

<br>

## Defaults
**Defaults**:
- if `mode` is **omitted**, the **default** is `RANGE`;
- if `frame_start` is **omitted**, the **default** is `UNBOUNDED PRECEDING`;
- if `frame_end` is **omitted**, the **default** is `CURRENT ROW`;
- if `PARTITION BY` is **omitted**, then **whole result** is a **single partition**;
- if `ORDER BY` is **omitted**, then **all rows in current partition** are **peers** of **current row**;
- `RANGE`, not `ROWS` (**current row** includes all peers);

<br>

Bringing it all together, the **default** *window frame definition*:
```sql
RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```
or **shorter version**:
```sql
RANGE UNBOUNDED PRECEDING
```

<br>

The **default** *window frame* also depends on `ORDER BY`:
- **with** `ORDER BY`, the **default frame** is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` (or *shorter version* `RANGE UNBOUNDED PRECEDING`);
  - in `RANGE` **mode** the `CURRENT ROW` means **not** only actually current row, instead the `CURRENT ROW` is a **set of all peer rows**;
  - a **peer row** is a row that has **equivalent** values in **all** columns specified in `ORDER BY` clause (aka **sorting criteria**) to the **current row**;
- **without** `ORDER BY` **all** rows of the *partition* are included in the *window frame*, since **all** rows become *peers* of the *current row*;
  - it is **equal to** `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`;

<br>

**Default** *window frames* in **different** modes:<br>
![win_funcs_default_window_frames](/img/win_funcs_default_window_frames.png)

<br>

Examples of **window frames definitions**:
![window_frame_definitions](/img/window_frame_definitions.png)

<br>

## Types of window functions
- **Aggregate**:
  - `avg()`
  - `count()`
  - `max()`
  - `min()`
  - `sum()`
- **Ranking**:
  - `row_number()`
  - `rank()`
    - firstly, rows are **sorted** by one or more **sorting columns**;
    - each row or group of rows that have the same values in the **sorting columns** is assigned a rank, the rank starts from 1;
    - if **several rows** have the **same values** in the **sorting columns**, they receive the **same rank**;
    - it **skips ranks**: after a group of rows with the same rank, the **next rank increases by the number of rows** in that group;
      - for example, if **two** rows have rank **2**, the **next** row **gets** rank **4**, **not** 3;
  - `dense_rank()`
    - unlike the `rank()` function, it **doesn't skip** ranks and after a group of identical values, the **next rank increases by one**, **not** by the number of rows;
      - for example, if **two** rows have rank **2**, the **next** row will get rank **3**, **not** 4;
- **Offset**:
  - `lag()`
    - accesses data **from previous rows** of the window;
    - it has **3** arguments:
      - the **column** whose value needs to be returned;
      - the **number of rows to offset** (**default** is **1**);
      - the **default** value to return if the offset returns a `NULL` value;
  - `lead()`
    - accesses data **from following rows** of the window;
    - it has **3** arguments:
      - the **column** whose value needs to be returned;
      - the **number of rows to offset** (**default** is **1**);
      - the **default** value to return if the offset returns a `NULL` value;
  - `first_value()`
    - returns the first value in the window;
  - `last_value()`
    - returns the last value in the window;

<br>

## Examples
### Example
Consider table:
|col_1|col_2||
|:-------|:-------|:-|
|a|1||
|a|2|current row|
|b|3||
|b|4|current row|
|b|5||

Consider query: `SELECT col_1 OVER (ORDER BY col_1)`. In this query `OVER (ORDER BY col_1)`.<br>
**Then**:
- the `CURRENT ROW` of the row `|b|4|` is a **set of rows**:
  - `|b|3|`
  - `|b|4|`
  - `|b|5|`
- the `CURRENT ROW` of the row `|a|2|` is a **set of rows**:
  - `|a|1|`
  - `|a|2|`

<br>

# Examples
## Case: get first row from every group
### CTE version
```sql
WITH summary AS (
    SELECT  id, 
            tstz, 
            operation, 
            ROW_NUMBER() OVER(PARTITION BY id ORDER BY tstz ASC) AS rk
    FROM logs)
SELECT s.*
  FROM summary s
 WHERE s.rk = 1;
```

<br>

### Subquery version
```sql
SELECT s.* FROM (
    SELECT  id, 
            tstz, 
            operation, 
            ROW_NUMBER() OVER(PARTITION BY id ORDER BY tstz ASC) AS rk
    FROM logs
) AS s
WHERE s.rk = 1;
```

<br>

## Case: get top N
### Preparation
```sql
DROP TABLE logs;
CREATE TABLE logs (uri VARCHAR(255) NOT NULL, src_ip INET NOT NULL);
```

<br>

```sql
INSERT INTO logs VALUES
    ('/auto', '1.1.1.1'),
    ('/auto', '2.2.2.2'),
    ('/music', '3.3.3.3'),
    ('/news', '3.3.3.3'),
    ('/news', '2.2.2.2'),
    ('/news', '1.1.1.1'),
    ('/news', '4.4.4.4'),
    ('/auto', '4.4.4.4'),
    ('/music', '5.5.5.5'),
    ('/news', '6.6.6.6'),
    ('/tech', '7.7.7.7'),
    ('/news', '7.7.7.7'),
    ('/finance', '8.8.8.8'),
    ('/medicine', '9.9.9.9'),
    ('/politics', '9.9.9.9'),
    ('/politics', '10.10.10.10'),
    ('/art', '10.10.10.10'),
    ('/art', '11.11.11.11'),
    ('/auto', '2.2.2.2'),
    ('/news', '4.4.4.4'),
    ('/music', '5.5.5.5'),
    ('/auto', '11.11.11.11'),
    ('/auto', '12.12.12.12'),
    ('/music', '13.13.13.13'),
    ('/news', '13.13.13.13'),
    ('/news', '12.12.12.12'),
    ('/news', '11.11.11.11'),
    ('/news', '14.14.14.14'),
    ('/auto', '14.14.14.14'),
    ('/music', '15.15.15.15'),
    ('/news', '16.16.16.16'),
    ('/tech', '17.17.17.17'),
    ('/news', '17.17.17.17'),
    ('/finance', '18.18.18.18'),
    ('/medicine', '19.19.19.19'),
    ('/politics', '19.19.19.19');
```

<br>

```sql
\d+ logs
                                           Table "public.logs"
 Column |          Type          | Collation | Nullable | Default | Storage  | Stats target | Description
--------+------------------------+-----------+----------+---------+----------+--------------+-------------
 uri    | character varying(255) |           | not null |         | extended |              |
 src_ip | inet                   |           | not null |         | main     |              |
Access method: heap
```

<br>

### Select TOP 2 src_ip per TOP 5 uri
#### CTE
```sql
WITH
    top_N_uri AS(
        SELECT uri, COUNT(*) AS uri_num
        FROM logs
        GROUP BY uri
        ORDER BY uri_num DESC
        LIMIT 5
    ),

    ip_hits_per_uri AS (
        SELECT l.uri, l.src_ip, COUNT(*) AS ip_num
        FROM logs l
        JOIN top_N_uri AS t
        ON l.uri = t.uri
        GROUP BY l.uri, l.src_ip
        -- ORDER BY l.uri, ip_num DESC
    ),

    enumerated_ip_hits_per_uri AS (
        SELECT t2.*, ROW_NUMBER() OVER (PARTITION BY uri ORDER BY t2.uri, t2.ip_num DESC, t2.src_ip) AS row_number
        FROM (
            SELECT l.uri, l.src_ip, COUNT(*) AS ip_num
            FROM logs AS l
            JOIN (
                SELECT uri, COUNT(*) AS uri_num
                FROM logs
                GROUP BY uri
                ORDER BY uri_num DESC
                LIMIT 5
            ) AS t1
            ON l.uri = t1.uri
            GROUP BY l.uri, l.src_ip
            -- ORDER BY l.uri, ip_num DESC
        ) AS t2
    )

    SELECT t.uri, t.src_ip FROM enumerated_ip_hits_per_uri t WHERE t.row_number <= 2;
```

<br>

#### Subqueries
```sql
SELECT t.uri, t.src_ip FROM (
    SELECT t2.*, ROW_NUMBER() OVER (PARTITION BY uri ORDER BY t2.uri, t2.ip_num DESC, t2.src_ip) AS row_number
    FROM (
        SELECT l.uri, l.src_ip, COUNT(*) AS ip_num
        FROM logs AS l
        JOIN (
            SELECT uri, COUNT(*) AS uri_num
            FROM logs
            GROUP BY uri
            ORDER BY uri_num DESC
            LIMIT 5
        ) AS t1
        ON l.uri = t1.uri
        GROUP BY l.uri, l.src_ip
        -- ORDER BY l.uri, ip_num DESC
    ) AS t2
) t
WHERE t.row_number <= 2;
```



<br>

## Case: get sales per country + city
### Preparation
```sql
DROP function IF EXISTS random_between;

CREATE OR REPLACE FUNCTION random_between(min bigint, max bigint) 
RETURNS bigint
AS $$
BEGIN
   RETURN floor(random()* (max-min + 1) + min);
END;
$$ language 'plpgsql' STRICT;

CREATE TABLE sales (fruit TEXT, country TEXT, city TEXT, price int4, date TIMESTAMPTZ);

INSERT INTO sales (fruit, country, city, price, date)
VALUES
    ('apple', 'Italy', 'Rome', random_between(10,1000), '01.01.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '01.01.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '01.05.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '01.05.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '01.10.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '01.10.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '01.15.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '01.15.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '01.20.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '01.20.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '01.25.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '01.25.2025 18:00'::timestamptz),

    ('apple', 'Italy', 'Rome', random_between(10,1000), '02.01.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '02.05.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '02.05.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '02.10.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '02.10.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '02.15.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '02.15.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '02.20.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '02.20.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '02.25.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '02.25.2025 18:00'::timestamptz),

    ('apple', 'Italy', 'Rome', random_between(10,1000), '03.01.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '03.05.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '03.05.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '03.10.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '03.10.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '03.15.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '03.15.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '03.20.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '03.20.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '03.25.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '03.25.2025 18:00'::timestamptz),

    ('apple', 'Italy', 'Rome', random_between(10,1000), '04.01.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '04.05.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '04.05.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '04.10.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '04.10.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '04.15.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '04.15.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '04.20.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '04.20.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '04.25.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '04.25.2025 18:00'::timestamptz),

    ('apple', 'Italy', 'Rome', random_between(10,1000), '05.01.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '05.05.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '05.05.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '05.10.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '05.10.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '05.15.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '05.15.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '05.20.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '05.20.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '05.25.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '05.25.2025 18:00'::timestamptz),

    ('apple', 'Italy', 'Rome', random_between(10,1000), '06.01.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '06.05.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '06.05.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '06.10.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '06.10.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '06.15.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '06.15.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '06.20.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '06.20.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '06.25.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Rome', random_between(10,1000), '06.25.2025 18:00'::timestamptz),

    ('apple', 'Italy', 'Milano', random_between(10,1000), '01.01.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '01.01.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '01.05.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '01.05.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '01.10.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '01.10.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '01.15.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '01.15.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '01.20.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '01.20.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '01.25.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '01.25.2025 18:00'::timestamptz),

    ('apple', 'Italy', 'Milano', random_between(10,1000), '02.01.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '02.05.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '02.05.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '02.10.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '02.10.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '02.15.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '02.15.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '02.20.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '02.20.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '02.25.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '02.25.2025 18:00'::timestamptz),

    ('apple', 'Italy', 'Milano', random_between(10,1000), '03.01.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '03.05.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '03.05.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '03.10.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '03.10.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '03.15.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '03.15.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '03.20.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '03.20.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '03.25.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '03.25.2025 18:00'::timestamptz),

    ('apple', 'Italy', 'Milano', random_between(10,1000), '04.01.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '04.05.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '04.05.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '04.10.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '04.10.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '04.15.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '04.15.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '04.20.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '04.20.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '04.25.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '04.25.2025 18:00'::timestamptz),

    ('apple', 'Italy', 'Milano', random_between(10,1000), '05.01.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '05.05.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '05.05.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '05.10.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '05.10.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '05.15.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '05.15.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '05.20.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '05.20.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '05.25.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '05.25.2025 18:00'::timestamptz),

    ('apple', 'Italy', 'Milano', random_between(10,1000), '06.01.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '06.05.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '06.05.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '06.10.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '06.10.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '06.15.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '06.15.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '06.20.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '06.20.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '06.25.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Milano', random_between(10,1000), '06.25.2025 18:00'::timestamptz),

    ('apple', 'Italy', 'Turin', random_between(10,1000), '01.01.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '01.01.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '01.05.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '01.05.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '01.10.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '01.10.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '01.15.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '01.15.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '01.20.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '01.20.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '01.25.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '01.25.2025 18:00'::timestamptz),

    ('apple', 'Italy', 'Turin', random_between(10,1000), '02.01.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '02.05.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '02.05.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '02.10.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '02.10.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '02.15.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '02.15.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '02.20.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '02.20.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '02.25.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '02.25.2025 18:00'::timestamptz),

    ('apple', 'Italy', 'Turin', random_between(10,1000), '03.01.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '03.05.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '03.05.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '03.10.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '03.10.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '03.15.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '03.15.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '03.20.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '03.20.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '03.25.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '03.25.2025 18:00'::timestamptz),

    ('apple', 'Italy', 'Turin', random_between(10,1000), '04.01.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '04.05.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '04.05.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '04.10.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '04.10.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '04.15.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '04.15.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '04.20.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '04.20.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '04.25.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '04.25.2025 18:00'::timestamptz),

    ('apple', 'Italy', 'Turin', random_between(10,1000), '05.01.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '05.05.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '05.05.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '05.10.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '05.10.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '05.15.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '05.15.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '05.20.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '05.20.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '05.25.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '05.25.2025 18:00'::timestamptz),

    ('apple', 'Italy', 'Turin', random_between(10,1000), '06.01.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '06.05.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '06.05.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '06.10.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '06.10.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '06.15.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '06.15.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '06.20.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '06.20.2025 18:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '06.25.2025 14:00'::timestamptz),
    ('apple', 'Italy', 'Turin', random_between(10,1000), '06.25.2025 18:00'::timestamptz),

    ('banana', 'Italy', 'Rome', random_between(10,1000), '01.01.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '01.01.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '01.05.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '01.05.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '01.10.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '01.10.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '01.15.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '01.15.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '01.20.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '01.20.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '01.25.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '01.25.2025 18:00'::timestamptz),

    ('banana', 'Italy', 'Rome', random_between(10,1000), '02.01.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '02.05.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '02.05.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '02.10.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '02.10.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '02.15.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '02.15.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '02.20.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '02.20.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '02.25.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '02.25.2025 18:00'::timestamptz),

    ('banana', 'Italy', 'Rome', random_between(10,1000), '03.01.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '03.05.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '03.05.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '03.10.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '03.10.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '03.15.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '03.15.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '03.20.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '03.20.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '03.25.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '03.25.2025 18:00'::timestamptz),

    ('banana', 'Italy', 'Rome', random_between(10,1000), '04.01.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '04.05.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '04.05.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '04.10.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '04.10.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '04.15.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '04.15.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '04.20.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '04.20.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '04.25.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '04.25.2025 18:00'::timestamptz),

    ('banana', 'Italy', 'Rome', random_between(10,1000), '05.01.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '05.05.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '05.05.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '05.10.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '05.10.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '05.15.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '05.15.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '05.20.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '05.20.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '05.25.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '05.25.2025 18:00'::timestamptz),

    ('banana', 'Italy', 'Rome', random_between(10,1000), '06.01.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '06.05.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '06.05.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '06.10.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '06.10.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '06.15.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '06.15.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '06.20.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '06.20.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '06.25.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Rome', random_between(10,1000), '06.25.2025 18:00'::timestamptz),

    ('banana', 'Italy', 'Milano', random_between(10,1000), '01.01.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '01.01.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '01.05.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '01.05.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '01.10.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '01.10.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '01.15.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '01.15.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '01.20.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '01.20.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '01.25.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '01.25.2025 18:00'::timestamptz),

    ('banana', 'Italy', 'Milano', random_between(10,1000), '02.01.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '02.05.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '02.05.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '02.10.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '02.10.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '02.15.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '02.15.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '02.20.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '02.20.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '02.25.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '02.25.2025 18:00'::timestamptz),

    ('banana', 'Italy', 'Milano', random_between(10,1000), '03.01.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '03.05.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '03.05.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '03.10.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '03.10.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '03.15.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '03.15.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '03.20.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '03.20.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '03.25.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '03.25.2025 18:00'::timestamptz),

    ('banana', 'Italy', 'Milano', random_between(10,1000), '04.01.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '04.05.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '04.05.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '04.10.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '04.10.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '04.15.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '04.15.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '04.20.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '04.20.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '04.25.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '04.25.2025 18:00'::timestamptz),

    ('banana', 'Italy', 'Milano', random_between(10,1000), '05.01.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '05.05.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '05.05.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '05.10.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '05.10.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '05.15.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '05.15.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '05.20.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '05.20.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '05.25.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '05.25.2025 18:00'::timestamptz),

    ('banana', 'Italy', 'Milano', random_between(10,1000), '06.01.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '06.05.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '06.05.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '06.10.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '06.10.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '06.15.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '06.15.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '06.20.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '06.20.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '06.25.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Milano', random_between(10,1000), '06.25.2025 18:00'::timestamptz),

    ('banana', 'Italy', 'Turin', random_between(10,1000), '01.01.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '01.01.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '01.05.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '01.05.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '01.10.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '01.10.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '01.15.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '01.15.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '01.20.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '01.20.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '01.25.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '01.25.2025 18:00'::timestamptz),

    ('banana', 'Italy', 'Turin', random_between(10,1000), '02.01.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '02.05.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '02.05.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '02.10.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '02.10.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '02.15.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '02.15.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '02.20.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '02.20.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '02.25.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '02.25.2025 18:00'::timestamptz),

    ('banana', 'Italy', 'Turin', random_between(10,1000), '03.01.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '03.05.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '03.05.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '03.10.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '03.10.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '03.15.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '03.15.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '03.20.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '03.20.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '03.25.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '03.25.2025 18:00'::timestamptz),

    ('banana', 'Italy', 'Turin', random_between(10,1000), '04.01.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '04.05.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '04.05.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '04.10.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '04.10.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '04.15.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '04.15.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '04.20.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '04.20.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '04.25.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '04.25.2025 18:00'::timestamptz),

    ('banana', 'Italy', 'Turin', random_between(10,1000), '05.01.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '05.05.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '05.05.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '05.10.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '05.10.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '05.15.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '05.15.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '05.20.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '05.20.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '05.25.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '05.25.2025 18:00'::timestamptz),

    ('banana', 'Italy', 'Turin', random_between(10,1000), '06.01.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '06.05.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '06.05.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '06.10.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '06.10.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '06.15.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '06.15.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '06.20.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '06.20.2025 18:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '06.25.2025 14:00'::timestamptz),
    ('banana', 'Italy', 'Turin', random_between(10,1000), '06.25.2025 18:00'::timestamptz),

    ('orange', 'Italy', 'Rome', random_between(10,1000), '01.01.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '01.01.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '01.05.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '01.05.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '01.10.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '01.10.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '01.15.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '01.15.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '01.20.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '01.20.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '01.25.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '01.25.2025 18:00'::timestamptz),

    ('orange', 'Italy', 'Rome', random_between(10,1000), '02.01.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '02.05.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '02.05.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '02.10.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '02.10.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '02.15.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '02.15.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '02.20.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '02.20.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '02.25.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '02.25.2025 18:00'::timestamptz),

    ('orange', 'Italy', 'Rome', random_between(10,1000), '03.01.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '03.05.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '03.05.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '03.10.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '03.10.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '03.15.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '03.15.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '03.20.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '03.20.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '03.25.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '03.25.2025 18:00'::timestamptz),

    ('orange', 'Italy', 'Rome', random_between(10,1000), '04.01.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '04.05.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '04.05.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '04.10.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '04.10.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '04.15.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '04.15.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '04.20.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '04.20.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '04.25.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '04.25.2025 18:00'::timestamptz),

    ('orange', 'Italy', 'Rome', random_between(10,1000), '05.01.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '05.05.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '05.05.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '05.10.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '05.10.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '05.15.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '05.15.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '05.20.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '05.20.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '05.25.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '05.25.2025 18:00'::timestamptz),

    ('orange', 'Italy', 'Rome', random_between(10,1000), '06.01.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '06.05.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '06.05.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '06.10.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '06.10.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '06.15.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '06.15.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '06.20.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '06.20.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '06.25.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Rome', random_between(10,1000), '06.25.2025 18:00'::timestamptz),

    ('orange', 'Italy', 'Milano', random_between(10,1000), '01.01.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '01.01.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '01.05.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '01.05.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '01.10.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '01.10.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '01.15.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '01.15.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '01.20.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '01.20.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '01.25.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '01.25.2025 18:00'::timestamptz),

    ('orange', 'Italy', 'Milano', random_between(10,1000), '02.01.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '02.05.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '02.05.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '02.10.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '02.10.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '02.15.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '02.15.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '02.20.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '02.20.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '02.25.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '02.25.2025 18:00'::timestamptz),

    ('orange', 'Italy', 'Milano', random_between(10,1000), '03.01.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '03.05.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '03.05.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '03.10.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '03.10.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '03.15.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '03.15.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '03.20.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '03.20.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '03.25.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '03.25.2025 18:00'::timestamptz),

    ('orange', 'Italy', 'Milano', random_between(10,1000), '04.01.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '04.05.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '04.05.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '04.10.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '04.10.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '04.15.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '04.15.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '04.20.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '04.20.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '04.25.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '04.25.2025 18:00'::timestamptz),

    ('orange', 'Italy', 'Milano', random_between(10,1000), '05.01.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '05.05.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '05.05.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '05.10.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '05.10.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '05.15.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '05.15.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '05.20.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '05.20.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '05.25.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '05.25.2025 18:00'::timestamptz),

    ('orange', 'Italy', 'Milano', random_between(10,1000), '06.01.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '06.05.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '06.05.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '06.10.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '06.10.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '06.15.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '06.15.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '06.20.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '06.20.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '06.25.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Milano', random_between(10,1000), '06.25.2025 18:00'::timestamptz),

    ('orange', 'Italy', 'Turin', random_between(10,1000), '01.01.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '01.01.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '01.05.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '01.05.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '01.10.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '01.10.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '01.15.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '01.15.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '01.20.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '01.20.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '01.25.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '01.25.2025 18:00'::timestamptz),

    ('orange', 'Italy', 'Turin', random_between(10,1000), '02.01.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '02.05.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '02.05.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '02.10.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '02.10.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '02.15.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '02.15.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '02.20.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '02.20.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '02.25.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '02.25.2025 18:00'::timestamptz),

    ('orange', 'Italy', 'Turin', random_between(10,1000), '03.01.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '03.05.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '03.05.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '03.10.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '03.10.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '03.15.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '03.15.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '03.20.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '03.20.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '03.25.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '03.25.2025 18:00'::timestamptz),

    ('orange', 'Italy', 'Turin', random_between(10,1000), '04.01.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '04.05.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '04.05.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '04.10.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '04.10.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '04.15.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '04.15.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '04.20.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '04.20.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '04.25.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '04.25.2025 18:00'::timestamptz),

    ('orange', 'Italy', 'Turin', random_between(10,1000), '05.01.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '05.05.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '05.05.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '05.10.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '05.10.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '05.15.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '05.15.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '05.20.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '05.20.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '05.25.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '05.25.2025 18:00'::timestamptz),

    ('orange', 'Italy', 'Turin', random_between(10,1000), '06.01.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '06.05.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '06.05.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '06.10.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '06.10.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '06.15.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '06.15.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '06.20.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '06.20.2025 18:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '06.25.2025 14:00'::timestamptz),
    ('orange', 'Italy', 'Turin', random_between(10,1000), '06.25.2025 18:00'::timestamptz),

    ('orange', 'Russia', 'Moscow', random_between(10,1000), '01.01.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '01.01.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '01.05.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '01.05.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '01.10.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '01.10.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '01.15.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '01.15.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '01.20.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '01.20.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '01.25.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '01.25.2025 18:00'::timestamptz),

    ('orange', 'Russia', 'Moscow', random_between(10,1000), '02.01.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '02.05.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '02.05.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '02.10.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '02.10.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '02.15.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '02.15.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '02.20.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '02.20.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '02.25.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '02.25.2025 18:00'::timestamptz),

    ('orange', 'Russia', 'Moscow', random_between(10,1000), '03.01.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '03.05.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '03.05.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '03.10.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '03.10.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '03.15.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '03.15.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '03.20.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '03.20.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '03.25.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '03.25.2025 18:00'::timestamptz),

    ('orange', 'Russia', 'Moscow', random_between(10,1000), '04.01.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '04.05.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '04.05.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '04.10.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '04.10.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '04.15.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '04.15.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '04.20.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '04.20.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '04.25.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '04.25.2025 18:00'::timestamptz),

    ('orange', 'Russia', 'Moscow', random_between(10,1000), '05.01.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '05.05.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '05.05.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '05.10.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '05.10.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '05.15.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '05.15.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '05.20.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '05.20.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '05.25.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '05.25.2025 18:00'::timestamptz),

    ('orange', 'Russia', 'Moscow', random_between(10,1000), '06.01.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '06.05.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '06.05.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '06.10.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '06.10.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '06.15.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '06.15.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '06.20.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '06.20.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '06.25.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Moscow', random_between(10,1000), '06.25.2025 18:00'::timestamptz),

    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '01.01.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '01.01.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '01.05.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '01.05.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '01.10.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '01.10.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '01.15.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '01.15.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '01.20.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '01.20.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '01.25.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '01.25.2025 18:00'::timestamptz),

    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '02.01.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '02.05.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '02.05.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '02.10.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '02.10.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '02.15.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '02.15.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '02.20.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '02.20.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '02.25.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '02.25.2025 18:00'::timestamptz),

    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '03.01.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '03.05.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '03.05.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '03.10.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '03.10.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '03.15.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '03.15.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '03.20.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '03.20.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '03.25.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '03.25.2025 18:00'::timestamptz),

    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '04.01.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '04.05.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '04.05.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '04.10.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '04.10.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '04.15.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '04.15.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '04.20.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '04.20.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '04.25.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '04.25.2025 18:00'::timestamptz),

    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '05.01.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '05.05.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '05.05.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '05.10.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '05.10.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '05.15.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '05.15.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '05.20.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '05.20.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '05.25.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '05.25.2025 18:00'::timestamptz),

    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '06.01.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '06.05.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '06.05.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '06.10.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '06.10.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '06.15.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '06.15.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '06.20.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '06.20.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '06.25.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Krasnodar', random_between(10,1000), '06.25.2025 18:00'::timestamptz),

    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '01.01.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '01.01.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '01.05.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '01.05.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '01.10.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '01.10.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '01.15.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '01.15.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '01.20.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '01.20.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '01.25.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '01.25.2025 18:00'::timestamptz),

    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '02.01.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '02.05.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '02.05.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '02.10.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '02.10.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '02.15.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '02.15.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '02.20.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '02.20.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '02.25.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '02.25.2025 18:00'::timestamptz),

    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '03.01.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '03.05.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '03.05.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '03.10.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '03.10.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '03.15.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '03.15.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '03.20.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '03.20.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '03.25.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '03.25.2025 18:00'::timestamptz),

    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '04.01.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '04.05.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '04.05.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '04.10.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '04.10.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '04.15.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '04.15.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '04.20.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '04.20.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '04.25.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '04.25.2025 18:00'::timestamptz),

    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '05.01.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '05.05.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '05.05.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '05.10.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '05.10.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '05.15.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '05.15.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '05.20.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '05.20.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '05.25.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '05.25.2025 18:00'::timestamptz),

    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '06.01.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '06.05.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '06.05.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '06.10.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '06.10.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '06.15.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '06.15.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '06.20.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '06.20.2025 18:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '06.25.2025 14:00'::timestamptz),
    ('orange', 'Russia', 'Tomsk', random_between(10,1000), '06.25.2025 18:00'::timestamptz),

    ('apple', 'Russia', 'Moscow', random_between(10,1000), '01.01.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '01.01.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '01.05.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '01.05.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '01.10.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '01.10.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '01.15.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '01.15.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '01.20.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '01.20.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '01.25.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '01.25.2025 18:00'::timestamptz),

    ('apple', 'Russia', 'Moscow', random_between(10,1000), '02.01.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '02.05.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '02.05.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '02.10.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '02.10.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '02.15.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '02.15.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '02.20.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '02.20.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '02.25.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '02.25.2025 18:00'::timestamptz),

    ('apple', 'Russia', 'Moscow', random_between(10,1000), '03.01.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '03.05.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '03.05.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '03.10.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '03.10.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '03.15.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '03.15.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '03.20.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '03.20.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '03.25.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '03.25.2025 18:00'::timestamptz),

    ('apple', 'Russia', 'Moscow', random_between(10,1000), '04.01.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '04.05.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '04.05.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '04.10.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '04.10.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '04.15.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '04.15.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '04.20.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '04.20.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '04.25.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '04.25.2025 18:00'::timestamptz),

    ('apple', 'Russia', 'Moscow', random_between(10,1000), '05.01.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '05.05.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '05.05.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '05.10.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '05.10.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '05.15.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '05.15.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '05.20.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '05.20.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '05.25.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '05.25.2025 18:00'::timestamptz),

    ('apple', 'Russia', 'Moscow', random_between(10,1000), '06.01.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '06.05.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '06.05.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '06.10.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '06.10.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '06.15.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '06.15.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '06.20.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '06.20.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '06.25.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Moscow', random_between(10,1000), '06.25.2025 18:00'::timestamptz),

    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '01.01.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '01.01.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '01.05.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '01.05.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '01.10.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '01.10.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '01.15.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '01.15.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '01.20.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '01.20.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '01.25.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '01.25.2025 18:00'::timestamptz),

    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '02.01.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '02.05.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '02.05.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '02.10.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '02.10.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '02.15.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '02.15.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '02.20.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '02.20.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '02.25.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '02.25.2025 18:00'::timestamptz),

    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '03.01.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '03.05.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '03.05.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '03.10.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '03.10.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '03.15.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '03.15.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '03.20.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '03.20.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '03.25.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '03.25.2025 18:00'::timestamptz),

    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '04.01.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '04.05.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '04.05.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '04.10.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '04.10.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '04.15.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '04.15.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '04.20.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '04.20.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '04.25.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '04.25.2025 18:00'::timestamptz),

    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '05.01.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '05.05.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '05.05.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '05.10.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '05.10.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '05.15.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '05.15.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '05.20.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '05.20.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '05.25.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '05.25.2025 18:00'::timestamptz),

    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '06.01.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '06.05.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '06.05.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '06.10.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '06.10.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '06.15.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '06.15.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '06.20.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '06.20.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '06.25.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Krasnodar', random_between(10,1000), '06.25.2025 18:00'::timestamptz),

    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '01.01.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '01.01.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '01.05.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '01.05.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '01.10.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '01.10.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '01.15.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '01.15.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '01.20.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '01.20.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '01.25.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '01.25.2025 18:00'::timestamptz),

    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '02.01.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '02.05.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '02.05.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '02.10.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '02.10.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '02.15.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '02.15.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '02.20.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '02.20.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '02.25.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '02.25.2025 18:00'::timestamptz),

    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '03.01.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '03.05.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '03.05.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '03.10.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '03.10.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '03.15.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '03.15.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '03.20.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '03.20.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '03.25.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '03.25.2025 18:00'::timestamptz),

    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '04.01.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '04.05.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '04.05.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '04.10.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '04.10.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '04.15.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '04.15.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '04.20.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '04.20.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '04.25.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '04.25.2025 18:00'::timestamptz),

    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '05.01.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '05.05.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '05.05.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '05.10.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '05.10.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '05.15.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '05.15.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '05.20.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '05.20.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '05.25.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '05.25.2025 18:00'::timestamptz),

    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '06.01.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '06.05.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '06.05.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '06.10.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '06.10.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '06.15.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '06.15.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '06.20.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '06.20.2025 18:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '06.25.2025 14:00'::timestamptz),
    ('apple', 'Russia', 'Tomsk', random_between(10,1000), '06.25.2025 18:00'::timestamptz)
;
```

<br>

### Select
```sql
SELECT *, sum(price) OVER (PARTITION BY extract(month FROM date) ORDER BY country,city,fruit) AS sum_price FROM sales LIMIT 605;
```