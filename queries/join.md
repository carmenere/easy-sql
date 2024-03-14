# JOIN
The general syntax: `SELECT ... FROM T1 JOIN T2 [ ON join_condition ]`.


<br>

## Join Types
- If **more than one** table reference is **listed** in the `FROM` clause, the tables are **cross-joined**: `SELECT * FROM T1, T2;` is **equal** to `SELECT * FROM T1 CROSS JOIN T2;`.
- `FROM T1 CROSS JOIN T2` is **equivalent** to `FROM T1 INNER JOIN T2 ON TRUE` and it is also equivalent to `FROM T1, T2`.
  - **Cross join** gives **cartesian product** of whole rows, if the tables have `N` and `M` rows respectively, the result table will have `N * M` rows.
- `INNER JOIN` is **not** an **intersection**, it is behave like **intersection** for columns with UNIQUE constraint.
- `LEFT OUTER JOIN`:
  - First, an **inner join** is performed.
  - Then, for each row in `T1` that does not satisfy the join condition with any row in `T2`, a **joined row** is added with `null` values in columns of `T2`. Thus, the **joined table** always has **at least one** row for each row in `T1`.
- `RIGHT OUTER JOIN`. This is the **converse** of a **left join**: the result table will always have a row for each row in `T2`.
- `FULL OUTER JOIN`:
  - First, an **inner join** is performed.
  - Then,
    - for each row in `T1` that does not satisfy the join condition with any row in `T2`, a **joined row** is added with `null` values in columns of `T2`;
    - for each row of `T2` that does not satisfy the join condition with any row in `T1`, a **joined row** is added with `null` values in columns of `T1`;

<br>

## INNER vs. OUTER
The words `INNER` and `OUTER` are **optional** in all forms.<br>
`INNER` is the **default**.<br>
`LEFT`, `RIGHT`, and `FULL` imply an **outer join**.

<br>

## USING clause
The `USING` clause takes a comma-separated list of the **shared column names** and **forms** a **join condition** that includes an equality comparison for each one.<br>
For example, `... FROM T1 JOIN T2 USING (a, b)` is shorthand for `... FROM T1 JOIN T2 ON T1.a = T2.a AND T1.b = T2.b`.<br>

The output of `JOIN USING` **suppresses** **redundant** columns.

<br>

## NATURAL clause
The `NATURAL` clause forms a `USING` list consisting of all column names that appear in both input tables automatically. As with `USING`, these columns appear only once in the output table. If there are **no** common column names, `NATURAL JOIN` behaves like `JOIN ... ON TRUE`, producing a cross-product join.

<br>

## Subqueries
**Subqueries** **must** be enclosed in **parentheses** and **must** be assigned a table **alias** name.<br>
**Subqueries** **cannot reference** columns that specified out of **parentheses**.<br>

For example:
```sql
FROM (SELECT * FROM table1) AS alias_name
```

<br>

A subquery can also be a `VALUES` list:
```sql
FROM (VALUES ('anne', 'smith'), ('bob', 'jones'), ('joe', 'blow')) AS names(first, last)
```

<br>

## LATERAL subqueries
Subqueries appearing in `FROM` can be preceded by the key word `LATERAL`.<br>
This **allows** them to **reference** columns provided by **preceding** `FROM` items.<br>

<br>

### Example
```sql
SELECT * FROM foo, LATERAL (SELECT * FROM bar WHERE bar.id = foo.bar_id) t;
```

<br>

## LATERAL JOIN
The difference between a **non-lateral** and a **lateral** `JOIN` lies in whether you can look to the **left hand** table's row.
A `LATERAL` join is more **like** a **correlated subquery**, but, **correlated subquery** can only return a **single** value, **not** multiple columns and not multiple rows.<br>

<br>

### Lateral `JOIN`
```sql
SELECT *
FROM table1 t1
CROSS JOIN LATERAL (
        SELECT  *
        FROM    t2
        WHERE   t1.col1 = t2.col1 -- t1.col1 only allowed because of lateral
) sub
```

### Non-lateral `JOIN`
```sql
SELECT *
FROM table1 t1
CROSS JOIN (
        SELECT  *
        FROM    t2
        WHERE   t2.col1 = 42 -- No reference to outer query
) sub
```

<br>

# ASOF JOIN
`ASOF JOIN` joins two different time-series.<br>
Consider 2 tables:<br>
```sql
SELECT ts, trunc(100*random()) AS val
FROM generate_series('2024-01-01'::TIMESTAMPTZ, '2024-01-31', '1 day') AS ts;
           ts           | val
------------------------+-----
 2024-01-01 00:00:00+03 |  99
 2024-01-02 00:00:00+03 |  67
 2024-01-03 00:00:00+03 |  49
 2024-01-04 00:00:00+03 |  87
 2024-01-05 00:00:00+03 |   4
 2024-01-06 00:00:00+03 |  32
 2024-01-07 00:00:00+03 |   5
 2024-01-08 00:00:00+03 |  58
 2024-01-09 00:00:00+03 |  26
 2024-01-10 00:00:00+03 |  70
 2024-01-11 00:00:00+03 |  23
 2024-01-12 00:00:00+03 |  36
 2024-01-13 00:00:00+03 |  15
 2024-01-14 00:00:00+03 |  99
 2024-01-15 00:00:00+03 |  93
 2024-01-16 00:00:00+03 |  63
 2024-01-17 00:00:00+03 |  44
 2024-01-18 00:00:00+03 |  15
 2024-01-19 00:00:00+03 |   0
 2024-01-20 00:00:00+03 |  78
 2024-01-21 00:00:00+03 |  95
 2024-01-22 00:00:00+03 |  54
 2024-01-23 00:00:00+03 |  70
 2024-01-24 00:00:00+03 |   1
 2024-01-25 00:00:00+03 |  99
 2024-01-26 00:00:00+03 |  19
 2024-01-27 00:00:00+03 |  29
 2024-01-28 00:00:00+03 |  41
 2024-01-29 00:00:00+03 |  62
 2024-01-30 00:00:00+03 |  41
 2024-01-31 00:00:00+03 |  59
(31 rows)
```

```sql
SELECT ts + INTERVAL '60 minutes' * random() AS ts, trunc(100*random()) AS val
FROM generate_series('2024-01-01'::TIMESTAMPTZ, '2024-01-31', '1 day') AS ts;
              ts               | val
-------------------------------+-----
 2024-01-01 00:49:58.609033+03 |  41
 2024-01-02 00:04:19.925478+03 |   0
 2024-01-03 00:40:31.944151+03 |  51
 2024-01-04 00:36:02.983339+03 |  40
 2024-01-05 00:37:26.249754+03 |  36
 2024-01-06 00:02:00.28237+03  |  68
 2024-01-07 00:44:24.459385+03 |  26
 2024-01-08 00:31:32.988046+03 |   1
 2024-01-09 00:35:09.445109+03 |  16
 2024-01-10 00:11:58.40908+03  |  25
 2024-01-11 00:08:19.776337+03 |  31
 2024-01-12 00:15:09.62811+03  |  65
 2024-01-13 00:41:51.610361+03 |  81
 2024-01-14 00:50:33.594576+03 |  45
 2024-01-15 00:12:59.785947+03 |  34
 2024-01-16 00:45:02.919268+03 |  61
 2024-01-17 00:39:11.402894+03 |   7
 2024-01-18 00:44:40.222441+03 |  72
 2024-01-19 00:53:52.125051+03 |  95
 2024-01-20 00:28:48.031935+03 |  97
 2024-01-21 00:02:40.782775+03 |  35
 2024-01-22 00:31:54.170234+03 |  99
 2024-01-23 00:26:39.463634+03 |  57
 2024-01-24 00:00:30.825269+03 |  14
 2024-01-25 00:33:48.308072+03 |  23
 2024-01-26 00:18:09.834834+03 |  58
 2024-01-27 00:58:07.060292+03 |  36
 2024-01-28 00:54:18.144196+03 |  25
 2024-01-29 00:11:55.704483+03 |  45
 2024-01-30 00:16:33.186616+03 |  62
 2024-01-31 00:13:47.133253+03 |  10
(31 rows)
```

<br>

Timestamps differ in both tables and `JOIN ON t1.ts = t2.ts` returns nothing:
```sql
SELECT * FROM
    (SELECT ts, trunc(100*random()) AS val
    FROM generate_series('2024-01-01'::TIMESTAMPTZ, '2024-01-31', '1 day') AS ts) AS t1
JOIN
    (SELECT ts + INTERVAL '60 minutes' * random() AS ts, trunc(100*random()) AS val
    FROM generate_series('2024-01-01'::TIMESTAMPTZ, '2024-01-31', '1 day') AS ts) AS t2
ON t1.ts = t2.ts;
```

<br>

The `ASOF JOIN` solves this problem: for each row in the `t1` it takes from the `t2` **exactly one row** that meets condition: `t2.ts <= t1.ts`.
Postgresql doesn't support `ASOF JOIN` but `JOIN LATERAL` also solves this problem:
```sql
SELECT * FROM
    (SELECT ts, trunc(100*random()) AS val
    FROM generate_series('2024-01-01'::TIMESTAMPTZ, '2024-01-31', '1 day') AS ts) AS t1
JOIN LATERAL
    (SELECT ts + INTERVAL '60 minutes' * random() AS ts, trunc(100*random()) AS val
    FROM generate_series('2024-01-01'::TIMESTAMPTZ, '2024-01-31', '1 day') AS ts
    WHERE ts <= t1.ts
    ORDER BY ts DESC LIMIT 1) AS t2
ON true;
```
