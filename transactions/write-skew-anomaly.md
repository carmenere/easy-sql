# Write skew anomaly
**Serialization anomaly** appears when the **result** of parallel execution of transactions **depends on order** in which transactions are commited.<br>
Example of **serialization anomaly** is **write skew anomaly**.<br>
Let's consider transactions that are executed **simultaneously**: **T1** and **T2**.<br>
Transaction `T1` updates **all** `white` **to** `black` and transaction `T2` **all** `black` **to** `white`:
**T1**:
```sql
-- white => black
UPDATE table
SET color = 'black'
WHERE color = 'white'
```

<br>

**T2**:
```sql
-- black => white
UPDATE table
SET color = 'white'
WHERE color = 'black'
```

Both transactions concurrently make **disjoint updates** (`T1` updates `white`, `T2` updates `black`), and finally **concurrently commit**, neither having seen the update performed by the other.<br>

The **different rows** are modified in both transactions, but the values in rows are being modified in one atransaction are used as **filter** for `UPDATE` in another.<br>
In other words, every transaction use condition that is affected by another transaction.<br>
If run concurrently at isolation level **lower** then **serializable level**, the result is that **all values** become **swapped**.<br>
If run concurrently at **serializable level**, the **first** transaction is ok, but the **second** is **aborted**.

<br>

## Repeatable read
**Transaction isolation level** is set to `repeatable read` in both transactions.<br>

**T1**
```sql
so_db=# begin;
BEGIN
so_db=*#
so_db=*# SET transaction isolation level repeatable read;
SET
so_db=*# update t set color='black' where color='white';
UPDATE 1
so_db=*# commit;
COMMIT
so_db=#
```

<br>

**T2**
```sql
so_db=# begin;
BEGIN
so_db=*# SET transaction isolation level repeatable read;
SET
so_db=*# update t set color='white' where color='black';
UPDATE 1
so_db=*# commit;
COMMIT
so_db=#
```

<br>

So, `repeatable read` **doesn't detect** *write skew anomaly*.

<br>

## Serializable
**Transaction isolation level** is set to `serializable` in both transactions.<br>
Any attempts to **serializable** would give off two **different results**: either T1's version or T2's version.<br>

**T1**
```sql
so_db=# begin;
BEGIN
so_db=*# SET transaction isolation level serializable;
SET
so_db=*#
so_db=*# update t set color='black' where color='white';
UPDATE 1
so_db=*# commit;
ERROR:  could not serialize access due to read/write dependencies among transactions
DETAIL:  Reason code: Canceled on identification as a pivot, during commit attempt.
HINT:  The transaction might succeed if retried.
so_db=#
```

<br>

**T2**
```sql
so_db=# begin;
BEGIN
so_db=*# SET transaction isolation level repeatable read;
SET
so_db=*# update t set color='white' where color='black';
UPDATE 1
so_db=*# commit;
COMMIT
so_db=# begin;
BEGIN
so_db=*# SET transaction isolation level serializable;
SET
so_db=*# update t set color='white' where color='black';
UPDATE 1
so_db=*# commit;
COMMIT
so_db=#
```

<br>

In transaction **T1** db returns **error**: `ERROR: could not serialize access due to read/write dependencies among transactions`.<br>
So, `serializable` **detects** *write skew anomaly*.