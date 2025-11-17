# Return result of query: `RETURN QUERY` and `RETURNS TABLE()`
```sql
DROP TABLE foo;
CREATE TABLE foo (id bigint, ts timestamptz);

DROP function IF EXISTS get_foo1;

CREATE OR REPLACE FUNCTION get_foo1(arg_id bigint) 
RETURNS TABLE(t timestamptz)
AS $$
BEGIN
    RETURN QUERY SELECT ts FROM foo WHERE id = arg_id;
END;
$$ language 'plpgsql';

SELECT get_foo1(1);
```

<br>

# Return scalar value from query: `SELECT %column% INTO result`
```sql
DROP TABLE foo;
CREATE TABLE foo (id bigint, ts timestamptz);

DROP function IF EXISTS get_foo2;

CREATE OR REPLACE FUNCTION get_foo2(arg_id bigint) 
RETURNS timestamptz
AS $$
DECLARE
  result timestamptz;
BEGIN
    SELECT ts INTO result FROM foo AS d WHERE id = arg_id;
    IF NOT FOUND THEN
        RETURN NULL;
    ELSE
        RETURN result;
    END IF;
END;
$$ language 'plpgsql';

SELECT get_foo2(1);
```
