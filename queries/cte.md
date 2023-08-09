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

<br>

# Insert parents and childs using CTE
```sql
DROP TABLE IF EXISTS child;
DROP TABLE IF EXISTS parent;

CREATE TABLE IF NOT EXISTS parent (
    id BIGSERIAL NOT NULL,
    name VARCHAR(255) NOT NULL,
    PRIMARY KEY (id)
);

CREATE TABLE IF NOT EXISTS child (
    id BIGSERIAL NOT NULL,
    pid bigint NOT NULL,
    name VARCHAR(255) NOT NULL,
    PRIMARY KEY (id),
    FOREIGN KEY (pid) REFERENCES parent(id) ON DELETE CASCADE
);
```

<br>

```sql
WITH 
    ins1 AS ( 
        INSERT INTO parent (name) 
        VALUES ('foo')
        -- ON CONFLICT DO NOTHING
        RETURNING *
    )
    INSERT INTO child (pid, name)
    SELECT id, 'child_name_1' FROM ins1
    RETURNING *;

WITH 
    ins1 AS ( 
        INSERT INTO parent (name) 
        VALUES ('baz')
        RETURNING *
    )
    INSERT INTO child (pid, name)
    SELECT id, 'child_name_2' FROM ins1
    RETURNING *;
```

<br>

```
SELECT * FROM parent;
SELECT * FROM child;
```