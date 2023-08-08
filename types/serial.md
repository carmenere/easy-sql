# SERIAL type
```sql
CREATE TABLE tbl1 (
    col1 SERIAL
);
```

is **equal** to:

```sql
CREATE SEQUENCE tbl1_foo1_seq;
CREATE TABLE tbl1 (
    col1 integer NOT NULL DEFAULT nextval('tbl1_foo1_seq')
);
ALTER SEQUENCE tbl1_foo1_seq OWNED BY tbl1.col1;
```

<br>

```sql
bar=# \d+ tbl1
                                                 Table "public.tbl1"
 Column |  Type   | Collation | Nullable |              Default               | Storage | Stats target | Description
--------+---------+-----------+----------+------------------------------------+---------+--------------+-------------
 col1   | integer |           | not null | nextval('tbl1_foo1_seq'::regclass) | plain   |              |
Access method: heap
```