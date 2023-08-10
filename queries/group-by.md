# GROUP BY
The `GROUP BY` clause **divides** the rows returned from the `SELECT` statement **into groups**.<br>

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

# Aggregate functions
For **each group**, you can apply an **aggregate function**: 
- `SUM(col)` calculates the **sum of values** in column `col` **per** the group;
- `COUNT(col)` calculates the **number of items** in column `col` **per** the group;
- `MIN(col)` finds **max value** in column `col` **per** the group;
- `MAX(col)` finds **min value** in column `col` **per** the group;
- `AVG(col)` calculates the **average of values** in column `col` **per** the group;

<br>

> **Note**:<br>
> All aggregate functions **ignore** `NULL` values.
> `COUNT(*)` **takes into account** `NULL` values.
> Aggregate functions can be applied **only** to **unique** values: `SELECT avg(DISTINCT price)`.

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
