# Recursive queries
A **recursive query** is a query that refers to a **recursive CTE**.<br>

```sql
WITH RECURSIVE t AS (
    /* Query 1 */ -- non-recursive term
    UNION [ALL]
    /* Query 2 */  -- recursive term
)
SELECT * FROM t;
```

<br>

A **recursive CTE** has three elements:
- **non-recursive term** is a query that forms the **base result set** of the CTE structure;
- **recursive term** is a query that is joined with the **non-recursive term** using the `UNION` or `UNION ALL` operator;
- **termination condition**: the **recursion stops** when **no** rows are returned from the **previous** iteration;

<br>

## Example
```sql
CREATE TABLE employees (
    employee_id serial PRIMARY KEY,
    full_name VARCHAR NOT NULL,
    manager_id INT
);

INSERT INTO employees (
    employee_id,
    full_name,
    manager_id
)
VALUES
    (1, 'Michael North', NULL),
    (2, 'Megan Berry', 1),
    (3, 'Sarah Berry', 1),
    (4, 'Zoe Black', 1),
    (5, 'Tim James', 1),
    (6, 'Bella Tucker', 2),
    (7, 'Ryan Metcalfe', 2),
    (8, 'Max Mills', 2),
    (9, 'Benjamin Glover', 2),
    (10, 'Carolyn Henderson', 3),
    (11, 'Nicola Kelly', 3),
    (12, 'Alexandra Climo', 3),
    (13, 'Dominic King', 3),
    (14, 'Leonard Gray', 4),
    (15, 'Eric Rampling', 4),
    (16, 'Piers Paige', 7),
    (17, 'Ryan Henderson', 7),
    (18, 'Frank Tucker', 8),
    (19, 'Nathan Ferguson', 8),
    (20, 'Kevin Rampling', 8);
```

<br>

The following query returns all subordinates of the manager with the id `2`:
```sql
WITH RECURSIVE subordinates AS (
    SELECT
        employee_id,
        manager_id,
        full_name
    FROM
        employees
    WHERE
        employee_id = 2

    UNION

    SELECT
        e.employee_id,
        e.manager_id,
        e.full_name
    FROM
        employees e
    INNER JOIN subordinates AS s ON s.employee_id = e.manager_id
)

SELECT * FROM  subordinates;
```