# Type casting
## int4range 
### int4range literal syntax
```sql
SELECT '[1, 2]'::int4range;
 int4range
-----------
 [1,3)
(1 row)
```

<br>

### int4range constructor syntax
```sql
SELECT int4range(1, 2, '[]');
 int4range
-----------
 [1,3)
(1 row)
```

<br>

## ARRAY of int
### ARRAY literal syntax
```sql
SELECT '{1, 2, 3}'::int[];
  int4
---------
 {1,2,3}
(1 row)
```

<br>

### ARRAY constructor syntax
```sql
SELECT array[1, 2, 3];
  array
---------
 {1,2,3}
(1 row)
```

<br>

### Unnested version

```sql
SELECT unnest(array[1, 2, 3]);
 unnest
--------
      1
      2
      3
(3 rows)
```

<br>

## Array of int4range
```sql
SELECT array['[1,2]'::int4range];
   array
-----------
 {"[1,3)"}
(1 row)
```

<br>

```sql
SELECT '{"[1,2]"}'::int4range[];
 int4range
-----------
 {"[1,3)"}
(1 row)
```

<br>

### Unnested version
```sql
SELECT unnest(array['[1,2]'::int4range, '[1,2]'::int4range]) AS ranges;
 ranges
--------
 [1,3)
 [1,3)
(2 rows)
```
