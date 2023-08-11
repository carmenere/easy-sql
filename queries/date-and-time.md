# Date and time
SQL requires the following syntax: `type literal` or `literal::type`.<br>
Remember that **any** `literal` needs to be enclosed in **single quotes**: `'value'`.<br>
Type `time` is **equivalent** of type `TIME WITHOUT TIME ZONE`.<br>
Valid input for the `TIMESTAMP` types consists of the **concatenation** of a **date** and a **time**, followed by an **optional time zone**.<br>
`TIMESTAMP` types can be easily casted to date and time types.<br>

|`type literal` syntax|`literal::type` syntax|
|:--------------------|:---------------------|
|`SELECT DATE '14/3/2005';`|`SELECT '14/3/2005'::date;`|
|`SELECT TIME WITHOUT TIME ZONE '18:30:10';`|`SELECT '18:30:10'::time;`|
|`SELECT TIME WITH TIME ZONE '18:30:10';`|`SELECT '18:30:10'::timetz;`|
|`SELECT TIME '18:30:10';`|`SELECT '18:30:10'::time;`|
|`SELECT TIMESTAMP WITH TIME ZONE '14/3/2005 18:30:10';`|`SELECT '14/3/2005 18:30:10'::timestamptz;`|
|`SELECT TIMESTAMP WITHOUT TIME ZONE '14/3/2005 18:30:10';`|`SELECT '14/3/2005 18:30:10'::timestamp;`|

<br>

There is parameter `datestyle (string)` to set the **display format** for date and time and **input format** for date values.<br>
This parameter contains **two** independent components:
- the **output format specification** (`ISO`, `Postgres`, `SQL`, or `German`);
- the **input specification** for *year*, *month* and *day* **ordering** (`DMY`, `MDY`, or `YMD`).

<br>

By default, `datestyle` is set to `ISO, MDY`.<br>
`ISO` is short for **ISO 8601**, example `1997-12-17 07:37:16-08`.<br>
The **SQL standard** requires the use of the `ISO 8601` **display format**.<br>

<br>

### Example
```sql
set datestyle to ISO, DMY;

bar=# SELECT date '20/10/2004';
    date
------------
 2004-10-20
(1 row)

bar=# SELECT date '10/20/2004';
ERROR:  date/time field value out of range: "10/20/2004"
LINE 1: SELECT date '10/20/2004';
                    ^
HINT:  Perhaps you need a different "datestyle" setting.
bar=#
```
<br>

# Time zones
The **SQL standard** differentiates `TIMESTAMP WITHOUT TIME ZONE` and `TIMESTAMP WITH TIME ZONE` literals by the presence of a `+` or `-` symbol and **time zone offset** after the time.<br>

A **time zone offset** of `+N` indicates that the  **local time** is **ahead** of **UTC** by `N` hours.<br>
A **time zone offset** of `-N` indicates that the  **local time** is `N` hours **behind UTC**.<br>

Consider example: `22:30+04`, here
- `22:30` is a **local time**;
- `04` is an **offset** from UTC;

So, UTC is equal `22:30 – 4 = 18:30`.

Hence, according to the standard:
- `TIMESTAMP '2004-10-19 10:23:54'` is a `TIMESTAMP WITHOUT TIME ZONE`;
- `TIMESTAMP '2004-10-19 10:23:54+02'` is a `TIMESTAMP WITH TIME ZONE`;

<br>

> **Note**<br>
> PostgreSQL will treat both of the above as `TIMESTAMP WITHOUT TIME ZONE`.<br>
> To ensure that a `literal` is treated as `TIMESTAMP WITH TIME ZONE`, give it the correct **explicit** type: `TIMESTAMP WITH TIME ZONE '2004-10-19 10:23:54+02'`.

<br>

## Internal representation
PostgreSQL uses the widely-used **Olson time zone database** for information about historical time zone rules.<br>
In a `literal` that has been determined to be `TIMESTAMP WITHOUT TIME ZONE`, PostgreSQL will **silently ignore** any time zone indication.<br>
For `TIMESTAMP WITH TIME ZONE`, the **internally stored value** is **always** in **UTC** (Universal Coordinated Time, traditionally known as Greenwich Mean Time, **GMT**).<br>

An input value that has an **explicit** time zone specified is **converted** to **UTC** using the **appropriate offset** for that time zone.<br>
If **no** time zone is stated in the input string, then it is assumed to be in the time zone indicated by the system's `timezone` parameter, and is **converted** to **UTC** using the offset for the system's `timezone`.<br>

When a value of type `TIMESTAMP WITH TIME ZONE` is output, it is **always converted** from **UTC** to the **current timezone zone**, and displayed as **local time** in that zone. To see the time in another time zone, either change timezone or use the `AT TIME ZONE` construct.<br>

Conversions between `TIMESTAMP WITHOUT TIME ZONE` and `TIMESTAMP WITH TIME ZONE` normally assume that the `TIMESTAMP WITHOUT TIME ZONE` value should be taken or given as timezone local time. A different time zone can be specified for the conversion using `AT TIME ZONE`.<br>

<br>

# Examples
## timezone
```sql
SET timezone = 'UTC';
SET timezone = 'Europe/Moscow';
```

<br>

## 

```sql
SET timezone = 'UTC';

CREATE TABLE ts_demo (ts TIMESTAMP, tstz TIMESTAMPTZ);

INSERT INTO ts_demo (ts, tstz) VALUES ('2017-01-01 00:00:00', '2017-01-01 00:00:00');

SELECT * FROM  ts_demo;
         ts          |          tstz
---------------------+------------------------
 2017-01-01 00:00:00 | 2017-01-01 00:00:00+00
(1 row)

SET timezone = 'America/New_York';

SELECT * FROM  ts_demo;
         ts          |          tstz
---------------------+------------------------
 2017-01-01 00:00:00 | 2016-12-31 19:00:00-05
(1 row)
```

<br>

### AT TIME ZONE
```sql
SELECT ts, tstz AT TIME ZONE 'MSK' FROM  ts_demo;
         ts          |      timezone
---------------------+---------------------
 2017-01-01 00:00:00 | 2017-01-01 03:00:00
(1 row)

SELECT ts, tstz AT TIME ZONE 'UTC' FROM  ts_demo;
         ts          |      timezone
---------------------+---------------------
 2017-01-01 00:00:00 | 2017-01-01 00:00:00
(1 row)
```

<br>

### Casting
```sql
SELECT ts::date, tstz::date FROM  ts_demo;
     ts     |    tstz
------------+------------
 2017-01-01 | 2016-12-31
(1 row)

SELECT ts::date, tstz::date, ts::time FROM  ts_demo;
     ts     |    tstz    |    ts
------------+------------+----------
 2017-01-01 | 2016-12-31 | 00:00:00
(1 row)
```
