# Temporary tables
A **temporary table**, as its name implied, is a **short-lived table**.<br>
PostgreSQL automatically **drops** the **temporary tables** *at the end* of a **session** or a **transaction**.<br>

```sql
CREATE TEMP TABLE my_table
(                                  
    id BIGINT NOT NULL,
    name VARCHAR(15) NOT NULL,
    prefix inet NOT NULL,
    EXCLUDE USING gist (id WITH =, prefix inet_ops WITH &&)
);
CREATE TABLE
```