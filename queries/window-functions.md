# Window functions
The general syntax:
```sql
SELECT window_func() OVER ( expression ) AS num
```

<br>

**Window function** `window_func` is applied to all rows inside **window** `OVER ( expression )`.<br>
**Empty window** `OVER ()` means that **whole result** is a **window**.<br>

<br>

## ORDER BY
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

# PARTITION BY
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

# PARTITION BY
```sql
SELECT * FROM (
    SELECT *, count(*)
    OVER
    (PARTITION BY
        name,
        status,
        comment
    ) AS count
    FROM tbl
) AS mytbl
WHERE mytbl.count > 1;
```

<br>

# Select first row from every group
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



SELECT s.* FROM (
    SELECT  id, 
            tstz, 
            operation, 
            ROW_NUMBER() OVER(PARTITION BY id ORDER BY tstz ASC) AS rk
    FROM logs
) AS s
WHERE s.rk = 1;
```

# TOP N
```sql
DROP TABLE logs;
CREATE TABLE logs (uri VARCHAR(255) NOT NULL, src_ip INET NOT NULL);

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
