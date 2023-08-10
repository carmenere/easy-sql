# GROUP BY
The `GROUP BY` clause **divides** the rows returned from the `SELECT` statement **into groups**.<br>
For **each group**, you can apply an **aggregate function**: 
- `SUM()` to calculate the **sum of items** in the groups;
- `COUNT()` to calculate the **number of items** in the groups.

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
