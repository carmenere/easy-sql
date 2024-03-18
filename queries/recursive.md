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
DO $$ BEGIN
    CREATE TYPE vkinds AS ENUM (
        'foo',
        'bar',
        'fizzbazz',
        'foobar'
    );
EXCEPTION
    WHEN duplicate_object THEN NULL;
END $$;

CREATE TABLE IF NOT EXISTS vertices (
    id BIGINT NOT NULL,
    kind vkinds NOT NULL,

    PRIMARY KEY (id, kind)
);

CREATE TABLE IF NOT EXISTS edges (
    id BIGSERIAL,
    -- lnode
    lid BIGINT NOT NULL,
    lkind vkinds NOT NULL,
    -- rnode
    rid BIGINT NOT NULL,
    rkind vkinds NOT NULL,

    FOREIGN KEY (lid, lkind) REFERENCES vertices(id, kind) ON DELETE CASCADE,
    FOREIGN KEY (rid, rkind) REFERENCES vertices(id, kind) ON DELETE CASCADE
);

INSERT INTO vertices (
    id,
    kind
)
VALUES
    (10, 'foo'::vkinds),
    (20, 'bar'::vkinds),
    (30, 'fizzbazz'::vkinds),
    (40, 'foo'::vkinds),
    (50, 'bar'::vkinds),
    (60, 'fizzbazz'::vkinds),
    (70, 'foo'::vkinds),
    (80, 'bar'::vkinds),
    (90, 'fizzbazz'::vkinds),
    (100, 'foobar'::vkinds);

INSERT INTO edges (
    id,
    lid,
    lkind,
    rid,
    rkind
)
VALUES
    (1, 10, 'foo'::vkinds, 20, 'bar'::vkinds),
    (2, 10, 'foo'::vkinds, 50, 'bar'::vkinds),
    (3, 40, 'foo'::vkinds, 20, 'bar'::vkinds),
    (4, 40, 'foo'::vkinds, 50, 'bar'::vkinds),
    (5, 40, 'foo'::vkinds, 80, 'bar'::vkinds),
    (6, 70, 'foo'::vkinds, 80, 'bar'::vkinds),
    (7, 20, 'bar'::vkinds, 30, 'fizzbazz'::vkinds),
    (8, 50, 'bar'::vkinds, 60, 'fizzbazz'::vkinds),
    (9, 50, 'bar'::vkinds, 90, 'fizzbazz'::vkinds),
    (10, 80, 'bar'::vkinds, 30, 'fizzbazz'::vkinds),
    (11, 80, 'bar'::vkinds, 60, 'fizzbazz'::vkinds),
    (12, 80, 'bar'::vkinds, 90, 'fizzbazz'::vkinds),
    (13, 60, 'fizzbazz'::vkinds, 100, 'foobar'::vkinds);
```

<br>

The following query returns all subordinates of the manager with the id `2`:
```sql
WITH RECURSIVE walks AS (
    SELECT
        e.id,
        e.lid,
        e.lkind,
        e.rid,
        e.rkind
    FROM
        edges e
    WHERE e.lid = 10

    UNION

    SELECT
        e.id,
        e.lid,
        e.lkind,
        e.rid,
        e.rkind
    FROM
        edges e
    INNER JOIN walks AS w ON w.rid = e.lid AND e.rkind IN ('foobar'::vkinds, 'fizzbazz'::vkinds)
)

SELECT * FROM  walks;
```