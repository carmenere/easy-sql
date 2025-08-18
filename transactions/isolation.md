<!-- TOC -->
* [Data consistency](#data-consistency)
* [The ANSI SQL isolation Levels](#the-ansi-sql-isolation-levels)
* [The concurrent anomalies](#the-concurrent-anomalies)
  * [P0: Dirty write](#p0-dirty-write)
  * [P1: Dirty read](#p1-dirty-read)
  * [P2: Non-repeatable read or Fuzzy read](#p2-non-repeatable-read-or-fuzzy-read)
  * [P3: Phantom read](#p3-phantom-read)
  * [P4: Lost update](#p4-lost-update)
  * [Read Skew](#read-skew)
  * [Write Skew](#write-skew)
    * [Write Skew: example](#write-skew-example)
* [Snapshot isolation vs. Serializability](#snapshot-isolation-vs-serializability)
* [Transaction isolation levels](#transaction-isolation-levels)
* [Examples](#examples)
  * [SET transaction isolation level for current session](#set-transaction-isolation-level-for-current-session)
  * [SHOW transaction isolation level for current session](#show-transaction-isolation-level-for-current-session)
    * [Current](#current)
    * [Default](#default)
  * [SET default transaction isolation level](#set-default-transaction-isolation-level)
    * [Through postgresql.conf](#through-postgresqlconf)
    * [Through ALTER DATABASE](#through-alter-database)
<!-- TOC -->

<br>

# Data consistency
The _key feature_ of relational database is their ability to _ensure_ **data consistency** or **data correctness**.<br>
At the database level it is possible to create **integrity constraints**, e.g. `UNIQUE`.<br>
If all required constraints can be formulated at the database level, consistency would be guaranteed. But some conditions are too complex for that.<br>
If app breaks consistency without breaking the integrity, there is no way for db to detect such violations.<br>
Thus, data _consistency_ is **stricter** than _integrity_.<br>

<br>

**Transaction** is a **set of operation** that has the following **properties** (aka **ACID**):
- **C**onsistency: it transforms database from one **consistent** state to another **consistent** state;
- **A**tomicity: **all operations** are executed as a **single unit** of work or rolled backed;
- **I**solation: it **doesn't affect other transactions**;
- **D**urability: after crash, the system may still contain some changes made by uncommitted transactions and **the system must be able to restore data consistency after craches**;

<br>

# The ANSI SQL isolation Levels
**Concurrent execution** of multiple correct **transactions** can lead to **several types of problems** (aka **concurrent phenomena**, **concurrent anomalies**, **read phenomena**) that violate the data consistency.<br>
In other words, **concurrent anomalies** are issues that can occur at concurrent reading and writing to the same data item in db.<br>

**Level of transaction isolation** defines the **degree** of how transaction can affect each other and how they can affect data consistency.<br>
The **ANSI SQL** (aka **SQL-92**) standard defines **4 levels of transaction isolation** in terms of **3 concurrent anomalies** they prevent.<br>
Every upper isolation level **fixes** some concurrent anomaly and **includes** all previous levels.<br>
Why there are several levels? To choose the right **balance** between data **consistency** and **perfomance**.<br>
Next level gives more consistency but decreases performance.<br>

<br>

The ANSI SQL **levels** and **anomalies**:

|Isolation Level|Dirty Read|Nonrepeatable Read|Phantom Read| All other anomalies |
|:--------------|:---------|:-----------------|:-----------|:--------------------|
|**Read uncommitted**|Possible, but **not in PG**|Possible|Possible| Possible            |
|**Read committed**|**Not possible**|Possible|Possible| Possible            |
|**Repeatable read**|**Not possible**|**Not possible**|Possible, but **not in PG**| Possible            |
|**Serializable**|**Not possible**|**Not possible**|**Not possible**| **Not possible**    |

<br>

# The concurrent anomalies
## P0: Dirty write
**Scenario**:
1. Transaction `T1` modifies a row.
2. Another transaction `T2` then further **modifies** that row **before** `T1` performs a `COMMIT` or `ROLLBACK`.
3. If `T1` or `T2` then performs a `ROLLBACK`, it is unclear what the correct data value should be.

The ANSI SQL isolation should be modified to require P0 for all isolation levels.
Actually P0 is fixed by all 

<br>

## P1: Dirty read
**Scenario**:
1. Transaction `T1` modifies a row.
2. Another transaction `T2` then **reads** that row **before** `T1` performs a `COMMIT` or `ROLLBACK`.
3. If `T1` then performs a `ROLLBACK`, `T2` has read a row that was never committed and so never really existed.

<br>

## P2: Non-repeatable read or Fuzzy read
**Scenario**:
1. Transaction `T1` reads a row.
2. Another transaction `T2` then **modifies** or **deletes** that row and `COMMIT`.
3. If `T1` then attempts to **reread** the row, it receives a **modified value** or discovers that the row has been **deleted**.

<br>

## P3: Phantom read
**Scenario**:
1. Transaction `T1` **reads** a **set of rows** satisfying some **search condition**.
2. Transaction `T2` then **creates** rows that satisfy `T1`'s **search condition** and `COMMIT`.
3. If `T1` then reads with the same **search condition**, it gets a **set of rows** different from the first read.

<br>

## P4: Lost update
**Scenario**:
1. Transaction `T1` **reads** a row.
2. Transaction `T2` **reads** a row.
2. Then `T2` **updates** the row and `COMMIT`.
3. Then `T1` **updates** the row and `COMMIT`.
4. The changes of `T1` cancel changes of `T2`, they become **lost**. 

<br>

## Read Skew
**Scenario**:
1. Transaction `T1` **reads** `x`.
2. Transaction `T2` **updates** `x` and `y` to new values and `COMMIT`.
3. Then `T1` reads `y` and it **doesn't** **re**read `x`, i.e. it holds **previous** value of `x`.
4. As a result, `T1` may produce an **inconsistent state** as output: if there were a **constraint** between `x` and `y`, it might be **violated**, for example **overall sum**. 

<br>

> **Note**:
> **Non-repeatable read** is a form of **Read skew** where `x` = `y`.<br>

<br>

## Write Skew
**Scenario 1**:
1. Transaction `T1` **reads** `x` and `y`. 
2. Transaction `T2` **reads** `x` and `y`
3. Then `T2` **writes** `x`, and `COMMIT`. 
4. Then `T1` **writes** `y`, and `COMMIT`. 
5. If there were a **constraint** between `x` and `y`, it might be **violated**, for example **overall sum**.

<br>

### Write Skew: example
1. Consider table of doctors and business **requirement** that **at least one** of doctor can be on call (on duty).
2. Consider that there are 2 doctors that are is on duty:
```sql
example=# select * from doctors ;
 name  | on_call
-------+---------
 Alice | t
 Bob   | t
(2 rows)
```
3. Consider they decide to cancel their shifts simultaneously.
3.1. Alice transaction:
```sql
BEGIN TRANSACTION;
DO $$ BEGIN
  IF (SELECT COUNT(*) FROM doctors WHERE on_call = true) >= 1 THEN
    UPDATE doctors SET on_call = false WHERE name = 'Alice';
  END IF;
END $$;
COMMIT; 
```
3.2. Bob transaction:
```sql
BEGIN TRANSACTION;
DO $$ BEGIN
  IF (SELECT COUNT(*) FROM doctors WHERE on_call = true) >= 1 THEN
    UPDATE doctors SET on_call = false WHERE name = 'Bob';
  END IF;
END $$;
COMMIT; 
```
4. After both transaction has commited there are will **no doctor on duty** and **constraint will be violated**:
```sql
example=# select * from doctors ;
 name  | on_call
-------+---------
 Bob   | f
 Alice | f
(2 rows)
```

<br>

# Snapshot isolation vs. Serializability
**Snapshot isolation** has been adopted by plenty database management systems, such as Oracle, MySQL, PostgreSQL, Microsoft SQL Server (2005 and later).<br>
The main reason for its adoption is that it allows **better performance** than **serializability**.<br>
In practice **snapshot isolation** is implemented within **multiversion concurrency control** (**MVCC**).<br>
A transaction executing under **snapshot isolation** appears to operate on a **personal snapshot** of the database, taken **at the start of the transaction**.<br>
When the transaction concludes, it will successfully commit only if the values updated by the transaction have not been changed externally since the snapshot was taken.<br>
Such a **write–write conflict** will cause the transaction to abort.<br>

**Lost Update** is _allowed_ by _Read committed_, but **proscribed** by **Snapshot Isolation**.<br>

<br>

# Transaction isolation levels
**Serializable level** is the strongest isolation level. It guarantees that parallel execution of transactions behaves as if they get executed serially.<br>

<br>

In PostgreSQL, you can request any of the 4 standard transaction isolation levels, but internally **only three** distinct isolation levels are implemented.<br>

**PostgreSQL's Read uncommitted** mode behaves like **Read committed**.<br>

This is because it is the only sensible way to map the standard isolation levels to PostgreSQL's **multiversion concurrency control** architecture.

<br>

# Examples
## SET transaction isolation level for current session
```sql
so_rs=# BEGIN;
BEGIN

so_rs=# SET transaction isolation level read uncommitted;
SET

so_rs=# SET transaction isolation level read committed;
SET

so_rs=# SET transaction isolation level repeatable read;
SET

so_rs=# SET transaction isolation level serializable;
SET
```

<br>

## SHOW transaction isolation level for current session
### Current
```sql
so_rs=# SHOW transaction isolation level;
 transaction_isolation
-----------------------
 serializable
(1 row)
```

<br>

### Default
```sql
so_rs=# SHOW default_transaction_isolation;
 default_transaction_isolation
-------------------------------
 read committed
(1 row)
```

<br>

## SET default transaction isolation level
### Through postgresql.conf
```sh
$ grep transac /usr/local/var/postgresql@12/postgresql.conf
#default_transaction_isolation = 'read committed'
#default_transaction_read_only = off
#default_transaction_deferrable = off
#idle_in_transaction_session_timeout = 0	# in milliseconds, 0 is disabled
#max_locks_per_transaction = 64		# min 10
#max_pred_locks_per_transaction = 64	# min 10
					# (max_pred_locks_per_transaction
```

<br>

### Through ALTER DATABASE
```sql
so_rs=# ALTER DATABASE so_rs SET DEFAULT_TRANSACTION_ISOLATION TO 'serializable';
ALTER DATABASE

so_rs=# SHOW default_transaction_isolation;
 default_transaction_isolation
-------------------------------
 serializable
(1 row)
```