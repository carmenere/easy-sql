# Table of contents
<!-- TOC -->
- [Table of contents](#table-of-contents)
- [GROUP BY](#group-by)
- [HAVING](#having)
- [Aggregate functions](#aggregate-functions)
- [Window functions](#window-functions)
  - [Examples](#examples)
    - [Example](#example)
    - [ORDER BY](#order-by)
    - [PARTITION BY](#partition-by)
- [Select first row from every group](#select-first-row-from-every-group)
  - [CTE version](#cte-version)
  - [Subquery version](#subquery-version)
- [TOP N](#top-n)
  - [Preparations](#preparations)
  - [Select TOP 2 src\_ip per TOP 5 uri](#select-top-2-src_ip-per-top-5-uri)
    - [CTE](#cte)
    - [Subqueries](#subqueries)
- [Example](#example-1)
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
There 2 variant of syntax:
- *window* is **directly** defined in list of columns:
```sql
SELECT window_func(col_1) OVER ( PARTITION BY col_2 ORDER BY col_3 ) AS t
ORDER BY col_7;
```
- **reference** to *window defenition*:
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

Every *aggregate function* followed by `window_func(col_1) OVER ( PARTITION BY ... ORDER BY ... )` clause becomes **window function**.<br>
Unlike *aggregate functions* that **collapse rows** into a single result, **window functions** return a value **for each row** in the result set.<br>

<br>

The `OVER()` clause specifies the **window** on which the **window function** will operate.<br>
A **window** is a a **result set of rows** that are somehow **related**.<br>
**Empty window** `OVER ()` means that **whole result** is a **window**.<br>
**Window function** `window_func` is applied to all rows inside **window** `window_func(col_1) OVER ( PARTITION BY ... ORDER BY ... )`.<br>

<br>

The `PARTITION BY col_1 [, col_2, ... ]` clause inside `OVER ()` **divides** the *window* into **independent logical groups** (aka **partitions**) based on the values of one or more specified columns. In other words **all rows inside partiton** have the **same value** in columns, specified in `PARTITION BY`.<br>
The **window function** then operates **independently** within each of these *partitions*.<br>
The `PARTITION BY` clause is optional. If you **omit** the `PARTITION BY` clause, the **window function** will treat the **whole result** set as a **single partition**.<br>

<br>

The `ORDER BY col_1 [, col_2, ... ]` clause **inside** `OVER ()` specifies the **order of rows in each partition** to which the window function is applied.<br>
**Every partition** is **sorted independently**.<br>
This ordering is crucial for functions that depend on row order, such as **ranking functions** (`row_number()`, `rank()`) or **cumulative calculations** (`sum()`).<br>

<br>

The `ORDER BY` clause uses the `NULLS FIRST` or `NULLS LAST` option to specify whether **nullable** values should be **first** or **last** in the result set. The **default** is `NULLS LAST` option.<br>

<br>

**Example**: consider table `(name, age)`, then `PARTITION BY age` divides all rows of *result set* into **partitions** where **all rows inside partiton** have the **same value** in column `age`:
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

A **window frame** is a **subset** of rows in the **current partition** that are somehow **related** to the **current row**. The window frame is evaluated separately within each partition. The window functions are **applied** to **window frame**.<br>
In other words, a **window frame** refines which rows within that partition are included in the calculation.<br>
A **window frame** is always bound to **current row**.<br>

A **window frame** can be **explicitly defined** by `ROWS`, `RANGE`, and `GROUPS` clauses. They are all **optional**:
```sql
OVER ([PARTITION BY <columns>] [ORDER BY <columns>] [ROWS | RANGE BETWEEN <lower_bound> AND <upper_bound>])
```

The `lower_bound` in `ROWS | RANGE` clauses can be:
- `UNBOUNDED PRECEDING`: **all** rows **before** the *current row*;
- `n PRECEDING`: **n** rows **before** the *current row*;
- `CURRENT ROW`: **just** the *current row*;

The `upper_bound` in `ROWS | RANGE` clauses can be:
- `CURRENT ROW`: **just** the *current row*;
- `n FOLLOWING`: **n** rows **after** the *current row*;
- `UNBOUNDED FOLLOWING`: **all** rows **after** the *current row*;

<br>

It is possible to use a **shorter version** of the **window frame definition**:
- `UNBOUNDED PRECEDING` is the same as `BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`;
- `UNBOUNDED FOLLOWING` is the same as `BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING`;
- `n PRECEDING` is the same as `BETWEEN n PRECEDING AND CURRENT ROW`, **n** is an integer value;
- `n FOLLOWING` is the same as `BETWEEN CURRENT ROW AND n FOLLOWING`, **n** is an integer value;

<br>

The **default** *window frame* definition depends on `ORDER BY`:
- without `ORDER BY`, the **default frame** is `RANGE UNBOUNDED PRECEDING`, which is the same as `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`;
  - this means **all** rows of the *partition* are included in the *window frame*, since **all** rows become *peers* of the *current row*;
- with `ORDER BY`, the **default frame** is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`;
  - in `RANGE` **mode** the `CURRENT ROW` means **not** only actually current row, instead the `CURRENT ROW` is a **set of all peer rows**;
  - a **peer row** is a row that has **equivalent** values in **all** columns specified in `ORDER BY` clause (aka **sorting criteria**) to the **current row**;
- in `ROWS` **mode**, `CURRENT ROW` is actually the **current row**;

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

### ORDER BY
- `ORDER BY` **at the end** of query sorts the **final** result;
- `ORDER BY` **inside window** determines **order** of appling **window function** `row_number()` to rows inside **window**;

<br>

```sql
SELECT
    id,
    section,
    header,
    score,
    row_number() OVER (ORDER BY score DESC)  AS rating
FROM news
ORDER BY id;
```

<br>

### PARTITION BY
- `PARTITION BY [expression]` inside window splits it into partitions and applies **window function** to every partition independently.<br>
- if `PARTITION BY [expression]` is **not** specified then the **whole window** is **partition**.<br>

<br>

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

# Select first row from every group
## CTE version
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

## Subquery version
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

# TOP N
## Preparations
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

## Select TOP 2 src_ip per TOP 5 uri
### CTE
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

### Subqueries
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




# Example
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
