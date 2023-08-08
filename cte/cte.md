# Common table expression
A **common table expression** (aka **CTE**) is a **temporary result** which you can use within another SQL statement.<br>
**CTEs** are typically used to **simplify** **complex joins** and **subqueries** in PostgreSQL.<br>

<br>

General **CTE** syntax:
```sql
WITH 
    cte_1 (column_list) AS (
        SELECT ...  
    ),
    ...
    cte_N (column_list) AS (
        UPDATE ...  
    )

SELECT ... / UPDATE ... FROM ... / DELETE ... ;
```
