# Functions to generate various random values
## Value of `boolean`
```sql
DROP function IF EXISTS random_bool;

CREATE OR REPLACE FUNCTION random_bool() 
RETURNS boolean
LANGUAGE sql
AS $$
   SELECT (round(random())::int)::boolean;
$$;
```

<br>

## Value of `bigint` between `min` and `max`
```sql
DROP function IF EXISTS random_between;

CREATE OR REPLACE FUNCTION random_between(min bigint, max bigint) 
RETURNS bigint
AS $$
BEGIN
   RETURN floor(random()* (max-min + 1) + min);
END;
$$ language 'plpgsql' STRICT;
```

<br>

## Array of `bigint`
```sql
DROP function IF EXISTS random_bigint_array;

CREATE OR REPLACE FUNCTION random_bigint_array(size bigint, min bigint, max bigint)
RETURNS bigint[]
LANGUAGE sql
AS $$
    SELECT array(SELECT random_between(min, max)
    FROM generate_series(1, trunc(random() * size + 1)::bigint));
$$;
```

<br>

## Random `text` with variable length between `min` and `max` and with custom prefix
```sql
DROP function IF EXISTS random_pstring;

CREATE OR REPLACE FUNCTION random_pstring(prefix text, min bigint, max bigint)
RETURNS TEXT
LANGUAGE sql
AS $$
    SELECT prefix || string_agg(substring('0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz', trunc(random() * 61 + 1)::int, 1), '')
    FROM generate_series(1, GREATEST(min, trunc(random() * max + 1)::bigint));
$$;
```

<br>

## Random `text`
```sql
DROP function IF EXISTS random_string;

CREATE OR REPLACE FUNCTION random_string(min bigint, max bigint)
RETURNS TEXT
LANGUAGE sql
AS $$
    SELECT string_agg(substring('0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz', trunc(random() * 61 + 1)::int, 1), '')
    FROM generate_series(1, GREATEST(min, trunc(random() * max + 1)::bigint));
$$;
```

<br>

## Resource ID `prefix-bigint`
```sql
DROP function IF EXISTS random_id;

CREATE OR REPLACE FUNCTION random_id(prefix text) RETURNS text
LANGUAGE sql
AS $$
    SELECT prefix || random_between(1, 9223372036854775807);
$$;
```

<br>

## Resource ID `prefix-bigint` between `min` and `max`
```sql
DROP function IF EXISTS random_id_between;

CREATE OR REPLACE FUNCTION random_id_between(prefix text, min bigint, max bigint) RETURNS text
LANGUAGE sql
AS $$
    SELECT prefix || random_between(min, max);
$$;
```

<br>

## IPv4
```sql
DROP function IF EXISTS random_ipv4;

CREATE OR REPLACE FUNCTION random_ipv4()
RETURNS inet
LANGUAGE sql
AS $$
    SELECT CONCAT(
        trunc(random() * 223 + 1), '.' , 
        trunc(random() * 250 + 2), '.', 
        trunc(random() * 250 + 2), '.',
        trunc(random() * 250 + 2), '/',
        trunc(random() * 31 + 1)
    )::inet;
$$;
```

<br>

# Date & Time
**Note**: values of `TIMESTAMPTZ` have following format `'2024-03-13 15:58:10.128086+03'`.<br>

## Convert `'2024-03-13 15:58:10.128086+03'` to UNIX (`bigint`)
```sql
DROP function IF EXISTS tstz_to_unix;

CREATE OR REPLACE FUNCTION tstz_to_unix(ts text)
RETURNS bigint
LANGUAGE sql
AS $$
   SELECT EXTRACT(EPOCH FROM ts::TIMESTAMPTZ)::bigint;
$$;
```

### Examples
```sql
SELECT tstz_to_unix('2024-01-01 00:00:00'); -- 1704056400

SELECT tstz_to_unix(
    (select '2024-01-01 00:00:00'::date + INTERVAL '1 day')::text
); -- 1704142800

SELECT int4range(1704056400, 1704142800-1, '[]') = int4range(1704056400, 1704142800, '[)'); -- t
```

<br>

## Random `timestamptz`
```sql
DROP function IF EXISTS random_timestamptz;

CREATE OR REPLACE FUNCTION random_timestamptz(min text, max text)
RETURNS timestamptz
LANGUAGE sql
AS $$
   SELECT to_timestamp(random_between(tstz_to_unix(min), tstz_to_unix(max)))::timestamptz;
$$;
```

<br>

## Random `interval`
```sql
DROP function IF EXISTS random_interval;

CREATE OR REPLACE FUNCTION random_interval(min bigint, max bigint)
RETURNS interval
LANGUAGE sql
AS $$
   SELECT (random_between(min, max) || ' min')::interval;
$$;
```

<br>

## Random `tstzrange`
```sql
DROP function IF EXISTS random_tstzrange;

CREATE OR REPLACE FUNCTION random_tstzrange(min text, max text)
RETURNS tstzrange
LANGUAGE plpgsql
AS $$
DECLARE
  lower timestamptz;
  upper timestamptz;
BEGIN
    lower = random_timestamptz(min, max);
    upper = random_timestamptz((lower + interval '1 sec')::text, max);
    RETURN(SELECT tstzrange(lower, t.id[trunc(random() * ARRAY_LENGTH(t.id, 1) + 1)], '[)')
    FROM (SELECT ARRAY[upper, '2030-01-01 00:00:00'::timestamptz] AS id) AS t);
END
$$;
```