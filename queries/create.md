# Create table
By default, every column is **nullable**.<br>

```sql
DROP TABLE IF EXISTS child;
DROP TABLE IF EXISTS parent;

CREATE TABLE IF NOT EXISTS parent (
    id BIGSERIAL NOT NULL,
    name VARCHAR(255) NOT NULL,
    active bool NOT NULL DEFAULT TRUE,
    PRIMARY KEY (id),
    CHECK ((char_length((name)::text) > 0))
);

CREATE TABLE IF NOT EXISTS child (
    id BIGSERIAL NOT NULL,
    pid bigint NOT NULL,
    name VARCHAR(255) NOT NULL,
    range int4range NOT NULL,
    PRIMARY KEY (id),
    FOREIGN KEY (pid) REFERENCES parent(id) ON DELETE CASCADE DEFERRABLE INITIALLY DEFERRED,
    UNIQUE (pid, name) DEFERRABLE INITIALLY DEFERRED,
    EXCLUDE USING gist (pid WITH =, range WITH &&),
    CONSTRAINT child_range_bounds CHECK (((lower(range) >= 0) AND (upper(range) < 1000))),
    CONSTRAINT child_forbid_empty_range CHECK ((range <> int4range(1, 1)))
);
```
