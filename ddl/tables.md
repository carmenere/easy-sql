# Table of contents
<!-- TOC -->
- [Table of contents](#table-of-contents)
- [CREATE table](#create-table)
- [DROP table](#drop-table)
- [Truncate table](#truncate-table)
- [Temporary tables](#temporary-tables)
- [ALTER table](#alter-table)
  - [Examples](#examples)
    - [ADD COLUMN](#add-column)
    - [DROP COLUMN](#drop-column)
    - [ADD CONSTRAINT](#add-constraint)
      - [UNIQUE](#unique)
      - [CHECK](#check)
      - [FOREIGN KEY](#foreign-key)
    - [RENAME CONSTRAINT](#rename-constraint)
    - [DROP CONSTRAINT](#drop-constraint)
    - [ADD PRIMARY KEY](#add-primary-key)
    - [ALTER COLUMN](#alter-column)
      - [SET NOT NULL](#set-not-null)
      - [DROP NOT NULL](#drop-not-null)
      - [DROP DEFAULT](#drop-default)
      - [Change type of column](#change-type-of-column)
  - [ADD CONSTRAINT ... PRIMARY KEY USING INDEX ...](#add-constraint--primary-key-using-index-)
<!-- TOC -->

<br>

# CREATE table
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

<br>

# DROP table
```sql
CREATE TABLE foo (id SERIAL PRIMARY KEY);
CREATE TABLE bar (
    pid INTEGER REFERENCES foo,
    name TEXT
);

demo=# \d+ bar
                                          Table "bookings.bar"
 Column |  Type   | Collation | Nullable | Default | Storage  | Compression | Stats target | Description
--------+---------+-----------+----------+---------+----------+-------------+--------------+-------------
 pid    | integer |           |          |         | plain    |             |              |
 name   | text    |           |          |         | extended |             |              |
Foreign-key constraints:
    "bar_pid_fkey" FOREIGN KEY (pid) REFERENCES foo(id)
Access method: heap

demo=# \d+ foo
                                                      Table "bookings.foo"
 Column |  Type   | Collation | Nullable |             Default             | Storage | Compression | Stats target | Description
--------+---------+-----------+----------+---------------------------------+---------+-------------+--------------+-------------
 id     | integer |           | not null | nextval('foo_id_seq'::regclass) | plain   |             |              |
Indexes:
    "foo_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "bar" CONSTRAINT "bar_pid_fkey" FOREIGN KEY (pid) REFERENCES foo(id)
Not-null constraints:
    "foo_id_not_null" NOT NULL "id"
Access method: heap
```

<br>

**Try to drop parnet table**:
```sql
DROP TABLE foo;
ERROR:  cannot drop table foo because other objects depend on it
DETAIL:  constraint bar_pid_fkey on table bar depends on table foo
HINT:  Use DROP ... CASCADE to drop the dependent objects too.
Time: 2.150 ms
```

**Drop with CASCADE**:
```sql
DROP TABLE foo CASCADE;

demo=# \d+ foo
Did not find any relation named "foo".
demo=# \d+ bar
                                          Table "bookings.bar"
 Column |  Type   | Collation | Nullable | Default | Storage  | Compression | Stats target | Description
--------+---------+-----------+----------+---------+----------+-------------+--------------+-------------
 pid    | integer |           |          |         | plain    |             |              |
 name   | text    |           |          |         | extended |             |              |
Access method: heap

demo=#
```

<br>

So, `DROP TABLE foo CASCADE` deletes **FK** constraint from **child** tables, but **child** tables **continue to exist**.<br>

<br>

# Truncate table
```sql
TRUNCATE foo;
DELETE FROM foo;
```

`TRUNCATE` is faster then `DELETE FROM`.<br>

<br>

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

<br>

# ALTER table
- `ADD COLUMN`:
```sql
ALTER TABLE foo ADD COLUMN x INTEGER NOT NULL CHECK (x>0);
```
- `DROP COLUMN`:
```sql
ALTER TABLE foo DROP COLUMN x
```
- `ALTER COLUMN` with **automatic** data conversion to new type:
```sql
ALTER TABLE foo ALTER COLUMN x SET DATA TYPE text;
```
- `ALTER COLUMN` with **explicit** data conversion to new type, asume that column `x` was of `integer` type and we change it to `text`:
```sql
INSERT INTO foo (x) VALUES (10),(11);
ALTER TABLE foo ALTER COLUMN x SET DATA TYPE text
USING 
    CASE
        WHEN x = 10 THEN 'a'::text
        WHEN x = 11 THEN 'b'::text
        ELSE 'z'::text
    END
;
```
**But** if there is a **constraint**, for example `"foo_x_check" CHECK (x > 0)` the above query will **fail** with **error**:
`ERROR:  operator does not exist: text > integer`.<br>
- `ADD CONSTRAINT`:
```sql
CREATE TABLE foo (id SERIAL PRIMARY KEY);
CREATE TABLE bar (name TEXT);
ALTER TABLE bar ADD CONSTRAINT fk_pid FOREIGN KEY (pid) REFERENCES foo (id);
```
- `DROP CONSTRAINT`:
```sql
ALTER TABLE bar DROP CONSTRAINT fk_pid
```
- `ADD CHECK|UNIQUE|FOREIGN KEY .. REFERENCES ..`:
```sql
CREATE TABLE bar (name TEXT);
ALTER TABLE bar ADD CHECK (name <> 'abc');
```
- `RENAME COLUMN`:
```sql
ALTER TABLE foo RENAME COLUMN x TO y;
```
- `RENAME CONSTRAINT`:
```sql
ALTER TABLE foo RENAME CONSTRAINT foo_pkey TO foo_pkey_new;
```

<br>

## Examples
### ADD COLUMN
```sql
ALTER TABLE some_tbl
    ADD COLUMN id BIGSERIAL NOT NULL;
```

<br>

### DROP COLUMN
```sql
ALTER TABLE some_tbl
    DROP COLUMN id;
```

<br>

### ADD CONSTRAINT
#### UNIQUE
```sql
ALTER TABLE some_tbl
    ADD CONSTRAINT custom_name_pkey UNIQUE (id);
```

<br>

#### CHECK
```sql
ALTER TABLE some_tbl
    ADD CONSTRAINT some_tbl_check
    CHECK (
        (vals = ANY (ARRAY[1, 2, 3, 4, 5]))
    );
```

<br>

#### FOREIGN KEY
```sql
ALTER TABLE distributors 
    ADD CONSTRAINT some_fk FOREIGN KEY (col1) REFERENCES tbl2 (col2);
```

<br>

### RENAME CONSTRAINT
```sql
DO $$ BEGIN
    ALTER TABLE child RENAME CONSTRAINT child_forbid_empty_range TO forbid_empty_range;
EXCEPTION
    WHEN undefined_object THEN NULL;
END $$;
```

<br>

### DROP CONSTRAINT
```sql
ALTER TABLE ONLY tbl
    DROP CONSTRAINT tbl_excl;
```

<br>

### ADD PRIMARY KEY
```sql
ALTER TABLE some_tbl 
    ADD PRIMARY KEY (id);
```

<br>

### ALTER COLUMN
#### SET NOT NULL
```sql
ALTER TABLE some_tbl 
    ALTER COLUMN my_column SET NOT NULL;
```

<br>

#### DROP NOT NULL
```sql
ALTER TABLE ONLY fw_icmp_messages
    ALTER COLUMN code DROP NOT NULL;
```

<br>

#### DROP DEFAULT
```sql
ALTER TABLE some_tbl 
    ALTER COLUMN my_column DROP DEFAULT;
```

#### Change type of column
PostgreSQL allows you to convert the values of a column to the new ones while changing its data type by adding a `USING` clause as follows:<br>
1. Consider following `enum` types:
```sql
CREATE TYPE echo_reply_t AS ENUM (
    'echo_reply'
);

CREATE TYPE destination_unreachable_t AS ENUM (
    'network_unreachable',
    'host_unreachable',
    'protocol_unreachable',
    'port_unreachable',
    'fragmentation_needed',
    'source_route_failed',
    'network_unknown',
    'host_unknown',
    'source_host_isolated',
    'network_prohibited',
    'host_prohibited',
    'tos_network_unreachable',
    'tos_host_unreachable',
    'communication_prohibited',
    'host_precedence_violation',
    'precedence_cutoff'
);

CREATE TYPE redirect_t AS ENUM (
    'network_redirect',
    'host_redirect',
    'tos_network_redirect',
    'tos_host_redirect'
);

CREATE TYPE echo_request_t AS ENUM (
    'echo_request'
);

CREATE TYPE router_advertisement_t AS ENUM (
    'router_advertisement',
    'does_not_route_common_traffic'
);

CREATE TYPE router_solicitation_t AS ENUM (
    'router_solicitation'
);

CREATE TYPE time_exceeded_t AS ENUM (
    'ttl_zero_during_transit',
    'ttl_zero_during_reassembly'
);

CREATE TYPE parameter_problem_t AS ENUM (
    'ip_header_bad',
    'required_option_missing',
    'bad_length'
);

CREATE TYPE timestamp_request_t AS ENUM (
    'timestamp_request'
);

CREATE TYPE timestamp_reply_t AS ENUM (
    'timestamp_reply'
);
```

<br>

```sql
DO $$ BEGIN
EXECUTE (
    SELECT format('CREATE TYPE icmp_codes_t AS ENUM (%s)', string_agg(DISTINCT quote_literal(variants), ', '))
    FROM (
        SELECT variants FROM UNNEST(
                                    enum_range(NULL::echo_reply_t)::text[]
                                    || enum_range(NULL::destination_unreachable_t)::text[]
                                    || enum_range(NULL::redirect_t)::text[]
                                    || enum_range(NULL::echo_request_t)::text[]
                                    || enum_range(NULL::router_advertisement_t)::text[]
                                    || enum_range(NULL::router_solicitation_t)::text[]
                                    || enum_range(NULL::time_exceeded_t)::text[]
                                    || enum_range(NULL::parameter_problem_t)::text[]
                                    || enum_range(NULL::timestamp_request_t)::text[]
                                    || enum_range(NULL::timestamp_reply_t)::text[]
                                    
                            ) AS variants
        ) t
);
END $$;
```

<br>

Then, convert type of column `code` from `int` to `icmp_codes_t` and change its values to new appropriate values:
```sql
ALTER TABLE ONLY fw_icmp_messages 
    ALTER COLUMN code 
        TYPE icmp_codes_t 
        USING CASE 
            WHEN type = 0  AND code = 0  THEN 'echo_reply'::icmp_codes_t 
            WHEN type = 3  AND code = 0  THEN 'network_unreachable'::icmp_codes_t
            WHEN type = 3  AND code = 1  THEN 'host_unreachable'::icmp_codes_t
            WHEN type = 3  AND code = 2  THEN 'protocol_unreachable'::icmp_codes_t
            WHEN type = 3  AND code = 3  THEN 'port_unreachable'::icmp_codes_t
            WHEN type = 3  AND code = 4  THEN 'fragmentation_needed'::icmp_codes_t
            WHEN type = 3  AND code = 5  THEN 'source_route_failed'::icmp_codes_t
            WHEN type = 3  AND code = 6  THEN 'network_unknown'::icmp_codes_t
            WHEN type = 3  AND code = 7  THEN 'host_unknown'::icmp_codes_t
            WHEN type = 3  AND code = 8  THEN 'source_host_isolated'::icmp_codes_t
            WHEN type = 3  AND code = 9  THEN 'network_prohibited'::icmp_codes_t
            WHEN type = 3  AND code = 10 THEN 'host_prohibited'::icmp_codes_t
            WHEN type = 3  AND code = 11 THEN 'tos_network_unreachable'::icmp_codes_t
            WHEN type = 3  AND code = 12 THEN 'tos_host_unreachable'::icmp_codes_t
            WHEN type = 3  AND code = 13 THEN 'communication_prohibited'::icmp_codes_t
            WHEN type = 3  AND code = 14 THEN 'host_precedence_violation'::icmp_codes_t
            WHEN type = 3  AND code = 15 THEN 'precedence_cutoff'::icmp_codes_t
            WHEN type = 5  AND code = 0  THEN 'network_redirect'::icmp_codes_t
            WHEN type = 5  AND code = 1  THEN 'host_redirect'::icmp_codes_t 
            WHEN type = 5  AND code = 2  THEN 'tos_network_redirect'::icmp_codes_t
            WHEN type = 5  AND code = 3  THEN 'tos_host_redirect'::icmp_codes_t
            WHEN type = 8  AND code = 0  THEN 'echo_request'::icmp_codes_t 
            WHEN type = 9  AND code = 0  THEN 'router_advertisement'::icmp_codes_t
            WHEN type = 9  AND code = 16 THEN 'does_not_route_common_traffic'::icmp_codes_t
            WHEN type = 10 AND code = 0  THEN 'router_solicitation'::icmp_codes_t
            WHEN type = 11 AND code = 0  THEN 'ttl_zero_during_transit'::icmp_codes_t
            WHEN type = 11 AND code = 1  THEN 'ttl_zero_during_reassembly'::icmp_codes_t
            WHEN type = 12 AND code = 0  THEN 'ip_header_bad'::icmp_codes_t
            WHEN type = 12 AND code = 1  THEN 'required_option_missing'::icmp_codes_t
            WHEN type = 12 AND code = 2  THEN 'bad_length'::icmp_codes_t
            WHEN type = 13 AND code = 0  THEN 'timestamp_request'::icmp_codes_t
            WHEN type = 14 AND code = 0  THEN 'timestamp_reply'::icmp_codes_t
            ELSE NULL
        END;
```

<br>

## ADD CONSTRAINT ... PRIMARY KEY USING INDEX ...
How to replace an automatically created primary key index by another index.<br>

```sql
CREATE UNIQUE INDEX ON bookings(book_ref) INCLUDE (book_date);

BEGIN;

ALTER TABLE bookings
    DROP CONSTRAINT bookings_pkey CASCADE;

ALTER TABLE bookings
    ADD CONSTRAINT bookings_pkey PRIMARY KEY
    USING INDEX bookings_book_ref_book_date_idx; -- a new index

ALTER TABLE tickets
    ADD FOREIGN KEY (book_ref) REFERENCES bookings(book_ref);

COMMIT;
```
