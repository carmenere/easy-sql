# JOIN
The general syntax: `SELECT ... FROM T1 JOIN T2 [ ON join_condition ]`.


<br>

## Join Types
- `FROM T1 CROSS JOIN T2` is **equivalent** to `FROM T1 INNER JOIN T2 ON TRUE`. **Cross join** gives **cartesian product** of whole rows, if the tables have `N` and `M` rows respectively, the result table will have `N * M` rows.
- `INNER JOIN` is an **intersection**, i.e., for each row `R1` of `T1`, the **joined table** has a row for each row in `T2` that satisfies the join condition with `R1`.
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
For example, `... FROM T1 JOIN T2 USING (a, b)` is shorthand for `... FROM T1 JOIN T2 ON T1.a = T2.a AND T1.b = T2.b`.

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
A `LATERAL` join is more **like** a **correlated subquery**, but, **correlated subquery** can only return a **single** value, **not** multiple columns and not multiple rows.
