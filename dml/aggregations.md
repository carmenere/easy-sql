# Table of contents
<!-- TOC -->
- [Table of contents](#table-of-contents)
- [GROUP BY](#group-by)
- [HAVING](#having)
- [Aggregate functions](#aggregate-functions)
- [Window functions](#window-functions)
  - [Window frame](#window-frame)
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

For **each group**, you can apply an **aggregate function**: 
- `sum(col)` calculates the **sum of values** in column `col` **per** the group;
- `count(col)` calculates the **number of items** in column `col` **per** the group;
- `min(col)` finds **max value** in column `col` **per** the group;
- `max(col)` finds **min value** in column `col` **per** the group;
- `avg(col)` calculates the **average of values** in column `col` **per** the group;

<br>

> **Note**:<br>
> All aggregate functions **ignore** `NULL` values.
> `count(*)` **takes into account** `NULL` values.
> Aggregate functions can be applied **only** to **unique** values: `SELECT avg(DISTINCT price)`.

<br>

# Window functions
Every *function* `func(col_1)` followed by `OVER ( PARTITION BY ... ORDER BY ... )` clause becomes **window function**.<br>
Unlike *aggregate functions* that **collapse rows** into a single result, **window functions** return a value **for each row** in the result set.<br>

**Syntax** of *window functions*:
```sql
SELECT <window_func>(<table_field>)
OVER (
      [PARTITION BY <partition_columns>]
      [ORDER BY <sort_columns>]
      [ROWS|RANGE <window_frame_definition>]
)
```

The `OVER()`clause is used to define the **window**.<br>
A **window** is a **group of rows** that will be passed to the **window function**. The **window functions** are **applied** to **window**.<br>
**Empty** `OVER ()` means that **window** is the **whole resulting rows**.<br>

<br>

There 2 variant of syntax:
- *window* is **directly** defined in list of columns:
```sql
SELECT window_func(col_1) OVER ( PARTITION BY col_2 ORDER BY col_3 ) AS t
ORDER BY col_7;
```
- **reference** to *window defenition* `SELECT .. FROM .. WINDOW win AS (...) ...`:
```sql
SELECT
    window_func(col_1) OVER win AS t1,
    window_func(col_2) OVER win AS t2,
    window_func(col_3) OVER win AS t3
FROM foo
WINDOW win AS ( PARTITION BY col_4 ORDER BY col_5 )
ORDER BY col_9;
```

<br>

**Example**:
```sql
SELECT
    id,
    section,
    header,
    score,
    row_number() OVER (PARTITION BY section ORDER BY score DESC)  AS rating_in_section
FROM news
ORDER BY section, rating_in_section;
```

<br>

The `OVER` clause has 3 **optional** clauses to **customize** the **window**:
- `PARTITION BY <partition_columns>`
  - **divides** the *resulting rows* into **non-overlapping** subsets of rows (aka **partitions**);
  - each **partition** contains only rows with the **same** values in **all** columns, specified in `PARTITION BY`;
  - **window functions** are applied **separately** to each partition, as if each were a **separate** data set;
  - if you **omit** the `PARTITION BY` clause, the **window function** will treat the **whole result** set as a **single partition**;
- `ORDER BY <sort_columns>`
  - **sorts** rows in **each** partition **independently**;
  - the `ORDER BY` clause uses the `NULLS FIRST` or `NULLS LAST` option to specify whether **nullable** values should be **first** or **last** in the result set;
  - the **default** is `NULLS LAST` option;
- `ROWS|RANGE <window_frame_definition>`
  - defines how rows are included into **window frame**;
  - in other words, this parameter defines how many rows to include **before** and **after** the **current row** in the **window frame**;
  - the **window frame** can **change** from row to row;

<br>

**Example**. Consider table `(name, age)`, then `PARTITION BY age` divides all rows of *result set* into **partitions** where **all rows inside partiton** have the **same value** in column `age`:
- **partition 1** (`age` = **30**):
```

| a    |  30 |
| a    |  30 |
| b    |  30 |
| b    |  30 |
| c    |  30 |
| c    |  30 |
```
- **partition 2** (`age` = **25**):
```
| a    |  25 |
| a    |  25 |
| b    |  25 |
| b    |  25 |
| c    |  25 |
| c    |  25 |
```

<br>

## Window frame
A **window frame** is a **subset** of rows in the **current partition** that are somehow **related** to the **current row**.<br>
The **window frame** is evaluated **separately** within each partition. 
In other words, a **window frame** refines which rows within that partition are included in the calculation.<br>
A **window frame** is always bound to **current row**.<br>
The **window frame** can **change** from row to row.<br>

A `<window_frame_definition>` in `ROWS|RANGE <window_frame_definition>` is specified as `BETWEEN <lower_bound> AND <upper_bound>`:
- where the `lower_bound` of **window frame** can be:
  - `UNBOUNDED PRECEDING`: **all** rows **before** the *current row*;
  - `n PRECEDING`: **n** rows **before** the *current row*;
  - `CURRENT ROW`: **just** the *current row*;
- where thw `upper_bound` of **window frame** can be:
  - `CURRENT ROW`: **just** the *current row*;
  - `n FOLLOWING`: **n** rows **after** the *current row*;
  - `UNBOUNDED FOLLOWING`: **all** rows **after** the *current row*;

<br>

**Full syntax examples**:
```sql
RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
```

<br>

Possible **window frames definitions**:
- `UNBOUNDED PRECEDING` is the same as `BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`;
- `UNBOUNDED FOLLOWING` is the same as `BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING`;
- `n PRECEDING` is the same as `BETWEEN n PRECEDING AND CURRENT ROW`, **n** is an integer value;
- `n FOLLOWING` is the same as `BETWEEN CURRENT ROW AND n FOLLOWING`, **n** is an integer value;

<br>

Examples of **window frames definitions**:
![window_frame_definitions](/img/window_frame_definitions.png)

<br>

The `ROWS` and `RANGE` work differently:
- `ROWS`
  - in `ROWS` **mode**, `CURRENT ROW` is actually the **current row**;
  - it defines **window frames** based on the **physical position** of rows relative to the **current row**;
    - for example, `1 PRECEDING` means **one** row **before** the **current** one;
  - it makes **precise frame**;
- `RANGE`
  - in `RANGE` **mode** the `CURRENT ROW` means **not** only actually current row, instead the `CURRENT ROW` is a **set of all peer rows**;
  - a **peer row** is a row that has **equivalent** values in **all** columns specified in `ORDER BY` clause (aka **sorting columns**) to the **current row**;
  - it defines **window frames** based on **values of** `<sort_columns>` of `ORDER BY`;
  - it makes **dynamic frames**, i.e. frames defined with `RANGE` can vary depending on the data;

<br>

The **default** *window frame* definition depends on `ORDER BY`:
- **with** `ORDER BY`, the **default frame** is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` (or shorter version `RANGE UNBOUNDED PRECEDING`);
  - in `RANGE` **mode** the `CURRENT ROW` means **not** only actually current row, instead the `CURRENT ROW` is a **set of all peer rows**;
  - a **peer row** is a row that has **equivalent** values in **all** columns specified in `ORDER BY` clause (aka **sorting criteria**) to the **current row**;
- **without** `ORDER BY`, the **default frame** is `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`;
  - this means **all** rows of the *partition* are included in the *window frame*, since **all** rows become *peers* of the *current row*;
- in `ROWS` **mode**, `CURRENT ROW` is actually the **current row**;

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