# LIKE
The symbol `%` means **any number of any characters**: from **0** to **infinity**.<br>
The symbol `_` means **exactly 1 any character**.<br>

```sql
demo=# SELECT * FROM airports WHERE airport_name LIKE '_';
 airport_code | airport_name | city | country | coordinates | timezone
--------------+--------------+------+---------+-------------+----------
(0 rows)

demo=# SELECT * FROM airports WHERE airport_name LIKE '__';
 airport_code | airport_name | city |   country    |   coordinates    |    timezone
--------------+--------------+------+--------------+------------------+-----------------
 KBS          | Bo           | Bo   | Sierra Leone | (-11.761,7.9444) | Africa/Freetown
 XYE          | Ye           | Ye   | Myanmar      | (97.867,15.3)    | Asia/Rangoon
(2 rows)
```

<br>

Also pg supports operators to work with **POSIX regular expressions**.<br>

**Example**:
```sql
demo=# SELECT * FROM airports WHERE airport_name ~ '^(Aw|Bro)';
 airport_code | airport_name |    city     |    country    |    coordinates     |      timezone
--------------+--------------+-------------+---------------+--------------------+---------------------
 BHQ          | Broken Hill  | Broken Hill | Australia     | (141.472,-32.0014) | Australia/Adelaide
 BMA          | Bromma       | Stockholm   | Sweden        | (17.9417,59.3544)  | Europe/Stockholm
 BME          | Broome       | Broome      | Australia     | (122.232,-17.9447) | Australia/Perth
 CBO          | Awang        | Cotabato    | Philippines   | (124.21,7.1652)    | Asia/Manila
 LYN          | Bron         | Lyon        | France        | (4.9443,45.7272)   | Europe/Paris
 SDM          | Brown        | San Diego   | United States | (-116.98,32.5723)  | America/Los_Angeles
 YBT          | Brochet      | Brochet     | Canada        | (-101.679,57.8894) | America/Winnipeg
(7 rows)

Time: 7.390 ms
```