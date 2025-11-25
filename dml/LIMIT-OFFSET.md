# LIMIT n OFFSET m
```sql
demo=# SELECT * FROM airports WHERE airport_name ~ '^(Aw|Bro)' LIMIT 2 OFFSET 5;
 airport_code | airport_name |   city    |    country    |    coordinates     |      timezone
--------------+--------------+-----------+---------------+--------------------+---------------------
 SDM          | Brown        | San Diego | United States | (-116.98,32.5723)  | America/Los_Angeles
 YBT          | Brochet      | Brochet   | Canada        | (-101.679,57.8894) | America/Winnipeg
(2 rows)

Time: 7.871 ms
```