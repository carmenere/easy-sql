# Updating the same row simultaneously
Let's consider transactions that are executed **simultaneously**: **T1** and **T2**.<br>
The **same row** is modified in both transactions.

<br>

## Repeatable read
**Transaction isolation level** is set to `repeatable read` in both transactions.<br>

**T1**
```sql
my_db=# begin;
BEGIN
my_db=*# SET transaction isolation level repeatable read;
SET
my_db=*# UPDATE t SET color = 'white' WHERE id = 2;
UPDATE 1
my_db=*# commit;
COMMIT
my_db=#
```

<br>

**T2**
```sql
my_db=# begin;
BEGIN
my_db=*# SET transaction isolation level repeatable read;
SET
my_db=*# UPDATE t SET color = 'green' WHERE id = 2;
ERROR:  could not serialize access due to concurrent UPDATE
my_db=!# rollback;
ROLLBACK
my_db=#
```

<br>

In transaction **T2** db returns **error**: `ERROR:  could not serialize access due to concurrent update`.<br>
So, `repeatable read` **doesn't allows** modify **the same row** concurrently.

<br>

## Read committed
**Transaction isolation level** is set to `read committed` in both transactions.

<br>

**T1**
```sql
my_db=# begin;
BEGIN
my_db=*# show transaction_isolation;
 transaction_isolation
-----------------------
 read committed
(1 row)

my_db=*# UPDATE t SET color = 'white' WHERE id = 2;
UPDATE 1
my_db=*# commit;
COMMIT
my_db=#
```

<br>

**T2**
```sql
my_db=# begin;
BEGIN
my_db=*# show transaction_isolation;
 transaction_isolation
-----------------------
 read committed
(1 row)

my_db=*# UPDATE t SET color = 'green' WHERE id = 2;
UPDATE 1
my_db=*# commit;
COMMIT
my_db=#
```

<br>

**Both** transactions completed **successfully**.<br>
So, `read committed` **allows** modify **the same row** concurrently.
