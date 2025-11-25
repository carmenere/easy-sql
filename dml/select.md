# SELECT ... FROM (VALUES ...)
```sql
SELECT * FROM (
    VALUES (5, 2), (7, 3)
) as t(col1, col2);
 col1 | col2
------+------
    5 |    2
    7 |    3
(1 row)
```

<br>

# array_fill
`array_fill` returns an array initialized with **supplied value** and **dimensions**.<br>

```sql
SELECT array_fill((SELECT id FROM (VALUES (100)) as tbl(id)), ARRAY[5]);
      array_fill
-----------------------
 {100,100,100,100,100}
(1 row)
```

<br>

# SELECT unnest(array[])
```sql
SELECT unnest(
    array[
        (1::bigint, 'xxx'::varchar, ARRAY[1,2,3]),
        (2::bigint, 'yyy'::varchar, ARRAY[1,2,3]),
        (3::bigint, 'sss'::varchar, ARRAY[1,2,3,4,5]),
        (4::bigint, 'aaa'::varchar, ARRAY[1])
    ]) as t;

           t
-----------------------
 (1,xxx,"{1,2,3}")
 (2,yyy,"{1,2,3}")
 (3,sss,"{1,2,3,4,5}")
 (4,aaa,{1})
(4 rows)
```

<br>

```sql
SELECT * FROM unnest(
    array[
        (1::bigint, 'xxx'::varchar, ARRAY[1,2,3]),
        (2::bigint, 'yyy'::varchar, ARRAY[1,2,3]),
        (3::bigint, 'sss'::varchar, ARRAY[1,2,3,4,5]),
        (4::bigint, 'aaa'::varchar, ARRAY[1])
    ]
) AS t(id bigint, name varchar(255), col int[]);

 id | name |     col
----+------+-------------
  1 | xxx  | {1,2,3}
  2 | yyy  | {1,2,3}
  3 | sss  | {1,2,3,4,5}
  4 | aaa  | {1}
(4 rows)
```