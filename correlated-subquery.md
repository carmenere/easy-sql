# Normal vs. correlated subquery
- In **normal subquery** the **inner** query is executed **first** and the **outer** query is **always dependent** on **inner** query.
- In **correlated subquery** the **outer** query is executed **first** and the **inner** query is **always dependent** on **outer** query.

<br>

Execution steps of **correlated subquery**:
1. Executes the **outer** query.
2. **For each row** of **outer** query **inner** subquery is executed **once**.

<br>


## Examples
### Find all employees whose salary is above average for their department

```sql
SELECT employee_number, name
FROM employees emp
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department = emp.department);
```

<br>

In the above query the **inner** query has to be **re-executed** *for each employee*.

<br>
 
### Find all departments that do not have any employees
```sql
SELECT department_id, department_name
FROM departments d
WHERE NOT EXISTS (
    SELECT 1
    FROM employees
    WHERE department_id = d.department_id);
```

<br>

`EXISTS` operator tests for existence of rows in the result set of the subquery.