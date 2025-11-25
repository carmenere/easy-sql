# Table of contents
<!-- TOC -->
- [Table of contents](#table-of-contents)
- [Data consistency](#data-consistency)
- [The transaction isolation levels](#the-transaction-isolation-levels)
- [Serialization error](#serialization-error)
- [The concurrent anomalies](#the-concurrent-anomalies)
  - [P0: Dirty write](#p0-dirty-write)
  - [P1: Dirty read](#p1-dirty-read)
  - [P2: Non-repeatable read or Fuzzy read](#p2-non-repeatable-read-or-fuzzy-read)
  - [P3: Phantom read](#p3-phantom-read)
  - [P4: Lost update](#p4-lost-update)
  - [Read Skew](#read-skew)
  - [Write Skew](#write-skew)
    - [Write Skew: example](#write-skew-example)
  - [Read-only transaction anomaly](#read-only-transaction-anomaly)
- [Concurrency Control](#concurrency-control)
  - [Pessimistic lock vs. Optimistic lock](#pessimistic-lock-vs-optimistic-lock)
- [MVCC. SI. SSI](#mvcc-si-ssi)
- [Transaction isolation levels](#transaction-isolation-levels)
- [Examples](#examples)
  - [SET transaction isolation level for current session](#set-transaction-isolation-level-for-current-session)
    - [`SET`](#set)
    - [`BEGIN`](#begin)
  - [SHOW transaction isolation level for current session](#show-transaction-isolation-level-for-current-session)
    - [Current transaction isolation](#current-transaction-isolation)
    - [Default transaction isolation](#default-transaction-isolation)
  - [SET default transaction isolation level](#set-default-transaction-isolation-level)
    - [Through postgresql.conf](#through-postgresqlconf)
    - [Through ALTER DATABASE](#through-alter-database)
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

# The transaction isolation levels
**Concurrent execution** of multiple _correct_ **transactions** can lead to **several types of problems** (aka **concurrent phenomena**, **concurrent anomalies**, **read phenomena** or just **phenomena**) that **violate** the _data consistency_.<br>
In other words, **concurrent anomalies** are issues that can occur at **concurrent reading** and **writing** to the **same** data item in db.<br>

The **SQL standard** (**ANSI SQL**, **SQL-92**) defines **four levels** of **transaction isolation**.<br>
These levels are **defined in terms of phenomena** which **must not occur** at each level.<br>
Every upper isolation level **fixes** particular concurrent **anomaly** (**phenomenon**) and **includes all** previous levels.<br>
Next level gives **more** _consistency_ but **decreases** _performance_.<br>
Why there are several levels? To choose the right **balance** between data **consistency** and **perfomance**.<br>

<br>

The four **levels** of transaction isolation and **anomalies** which they fix:

|Isolation Level|Dirty Read|Nonrepeatable Read|Phantom Read| All other anomalies |
|:--------------|:---------|:-----------------|:-----------|:--------------------|
|**Read uncommitted**|Possible, but **not in PG**|Possible|Possible| Possible            |
|**Read committed**|**Not possible**|Possible|Possible| Possible            |
|**Repeatable read**|**Not possible**|**Not possible**|Possible, but **not in PG**| Possible            |
|**Serializable**|**Not possible**|**Not possible**|**Not possible**| **Not possible**    |

<br>

The **PostgreSQL** implementation of the **Repeatable read** isolation level prevents **all** the anomalies described in the standard. But **not** all possible ones.<br>
However, one important fact is proved for sure: snapshot isolation does not prevent only two anomalies:
- **write skew** anomaly;
- **read-only transaction** anomaly;

<br>

**All other anomalies** are also known as **serialization anomalies**.<br>
**Serialization anomalies** arise when the **result** of execution of concurrent transactions **depends** on **order** in which transactions are commited.<br>
Examples of _serialization anomaly_ are
- **write skew** anomaly;
- **read skew** anomaly;
- **lost update**;

<br>

The **Serializable level** is the strongest isolation level: it **fixes all possible anomalies**.<br>

<br>

# Serialization error
When **Repeatable read** or **Serializable** are used the DB can throw a **serialization error**:
- `ERROR: could not serialize access due to concurrent update` for **Repeatable read**;
- `ERROR: could not serialize access due to read/write dependencies among transactions` for **Serializable**;

<br>

So, **application** must be ready to handle a **serialization error**, in other words it must be able to **retry transactions** that have been completed with a serialization failure.<br>

<br>

# The concurrent anomalies
## P0: Dirty write
**Scenario**:
1. Transaction `T1` **modifies** a row.
2. Then transaction `T2` **modifies** that row **before** `T1` performs a `COMMIT` or `ROLLBACK`.
3. If `T1` or `T2` then performs a `ROLLBACK`, it is unclear what the correct data value should be.

Actually `P0` is **fixed** in **all levels**.<br>

<br>

> **Note**:<br>
> **Fixed** by **Read uncommitted**.<br>

<br>

## P1: Dirty read
The **dirty read** anomaly occurs when a transaction **sees uncommitted changes** made by another transaction.<br>

**Scenario**:
1. Transaction `T1` **updates** a row.
2. Then transaction `T2` **reads** that row **before** `T1` performs a `COMMIT` or `ROLLBACK`.
3. If `T1` then performs a `ROLLBACK`, `T2` has read a row that was never committed and so never really existed.

<br>

> **Note**:<br>
> **Fixed** by **Read committed**.<br>
> But **in PostgreSQL** it is also **fixed** by **Read uncommitted**.<br>
> In PostgreSQL **Read uncommitted** is **equivalent** for **Read committed**.<br>

<br>

## P2: Non-repeatable read or Fuzzy read
The **non-repeatable read** anomaly occurs when the **first** transaction **reads** _the same row_ **twice**, 
whereas the **second** transaction **updates** (or **deletes**) this row between these reads and **commits** the change.<br>
As a result, the **first** transaction **gets different results**.<br>

<br>

**Scenario**:
1. Transaction `T1` **reads** a row.
2. Then transaction `T2` then **updates** or **deletes** that row and `COMMIT`.
3. If `T1` then attempts to **reread** the row, it receives a **modified value** or discovers that the row has been **deleted**.

<br>

> **Note**:<br>
> When isolation level allows **non-repeatable read** (e.g. **Read committed**) then you must **not** take any decisions based on the data read by the previous operator, as everything can change in between.<br>

<br>

> **Note**:<br>
> **Fixed** by **Repeatable read**.<br>

<br>

## P3: Phantom read
The **phantom read** anomaly occurs when the **first** transaction **fetches** a **set of rows** that **satisfy** a particular **condition** (search condition),
while another transaction **inserts** or **deletes** some other rows satisfying that condition and commits the changes.<br>
As a result, the first transaction **gets two different sets of rows**.<br>

<br>

**Scenario**:
1. Transaction `T1` **reads** a **set of rows** that satisfy a particular condition (search condition).
2. Transaction `T2` then **creates** rows that satisfy `T1`'s **search condition** and `COMMIT`.
3. If `T1` then reads with the same **search condition**, it gets a **set of rows** different from the first read.

<br>

> **Note**:<br>
> **Fixed** by **Serializable**.<br>
> But **in PostgreSQL** it is also **fixed** by **Repeatable read**.<br>

<br>

## P4: Lost update
**Scenario**:
1. Transaction `T1` **reads** a row.
2. Transaction `T2` also **reads** that row. 
3. Then `T1` **updates** the row and `COMMIT`. 
4. Then `T2` **updates** the row and `COMMIT`. 
5. The changes of `T1` have **lost** because `T2` **didn't** take into account any changes made by the transaction `T1`.

<br>

## Read Skew
**Scenario 1**:
1. Transaction `T1` **reads** `x`.
2. Transaction `T2` **updates** `x` and `y` to new values and `COMMIT`.
3. Then `T1` reads `y` and it **doesn't eread** `x`, i.e. it holds **previous** value of `x`.
4. As a result, `T1` may produce an **inconsistent state** as output: if there were a **constraint** between `x` and `y`, it might be **violated**, for example **overall sum**. 

<br>

**Scenario 2**:
1. Transaction `T1` set **read commited** level.
2. Transaction `T2` set **read commited** level. 
3. Transaction `T1` **updates** `x`, but **doesn't** `COMMIT`. 
4. Transaction `T2` **reads** `x` and sees its **original** value, it **doesn't** see changes made by `T1`. 
5. Transaction `T1` **updates** `y` and `COMMIT`. 
6. Transaction `T2` **reads** `y` and sees its **updated** value.
7. So, `T2` has inconsistent data: it uses **original** value of `x` and **updated** value of `y`, the values `x` and `y` are **inconsistent**. 
8. As a result, `T2` may produce an **inconsistent state** as output: if there were a **constraint** between `x` and `y`, it might be **violated**, for example **overall sum**.


> **Note**:
> **Non-repeatable read** is a form of **read skew** where `x` = `y`.<br>

<br>

How can you avoid this anomaly at the **Read committed** level? The answer is obvious: use a **single operator**.<br>
**The data visibility can change only between operators**.<br>

<br>

## Write Skew
**Scenario 1**:
1. Transaction `T1` **reads** `x` and `y`. 
2. Transaction `T2` **reads** `x` and `y`
3. Then `T2` **writes** `x`, and `COMMIT`. 
4. Then `T1` **writes** `y`, and `COMMIT`. 
5. If there were a **constraint** between `x` and `y`, it might be **violated**, for example **overall sum**.

<br>

**Scenario 2**:
1. Consider a following **constraint** between rows `x` and `y`: **overall sum** of `x.column1` and `y.column1` **cannot** exceed some **threshold value**.
2. Transaction `T1` set **repeatable read** level. 
3. Transaction `T2` set **repeatable read** level. 
4. Transaction `T1` **reads** `x` and `y` and calculates their sum. 
5. Transaction `T2` **reads** `x` and `y` and calculates their sum. 
6. Transaction `T1` fairly assumes that it can **increase** `x` by `100`, because it **doesn't violate** constraint. 
7. Transaction `T2` also assumes that it can **increase** `y` by `200`, because it **doesn't violate** constraint.
8. Transaction `T1` updates `x` and `COMMIT`.
9. Transaction `T2` updates `y` and `COMMIT`. 
10. As a result, **threshold value** is exeeded and constraint is violated. The data is **inconsistent**.

<br>

So, **Repeateable read** cannot recognize such anomaly.<br>

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

## Read-only transaction anomaly
To observe this anomaly, we have to run **three** transactions: **two** of them are going to **update** the data, while the **third** one will be **read-only**.<br>

1. Consider a following **constraint** between rows `x` and `y`: **overall sum** of `x.column1` and `y.column1` **cannot** exceed some **threshold value**.
2. Transaction `T1` set **repeatable read** level.
3. Transaction `T2` set **repeatable read** level.
4. Transaction `T1` **updates** `x`.
5. Transaction `T2` **updates** `y` and `COMMIT`.
6. Transaction `T3` is **started** and set **repeatable read** level.
7. Transaction `T3` could see the changes made by the transaction `T2`, but not by the `T1` one (which had not been committed yet).
8. Transaction `T3` can see inconsistent state, because `T1` is not commited.

<br>

# Concurrency Control
## Pessimistic lock vs. Optimistic lock
**Pessimistic lock** is exclusive lock until work is finished.<br>
**Optimistic lock** is a **lock free approach**, based on versioning, if transaction sees that version was changed it is immediately aborted.<br>

<br>

**Pessimistic lock** is a good when the **cost** of **locks** is **less** than the **cost** of **rollbacks**.<br>
It is good in the environment with **high contention for data**, i.e. when multiple concurrent transaction changes the same data.<br>

**Optimistic lock** is a good when the **cost** of **rollbacks** is **less** than the **cost** of **rolling back**.<br>
It is good in the environment with **low contention for data**.<br>

<br>

# MVCC. SI. SSI
There are three broad concurrency control techniques:
- **MVCC**: Multi-version Concurrency Control;
- **S2PL**: Strict Two-Phase Locking;
- **OCC**: Optimistic Concurrency Control;

The main **advantage** of **MVCC** is that **readers don’t block writers**, and **writers don’t block readers**.<br>
PostgreSQL use two variations of MVCC called **SI** (**Snapshot Isolation**) and **SSI** (Serializable Snapshot Isolation). The **SSI** provide **serializable isolation level**.<br>

<br>

> **Note**:<br>
> PostgreSQL uses **SSI** for **DML** (Data Manipulation Language, e.g, SELECT, UPDATE, INSERT, DELETE).<br>
> PostgreSQL uses **2PL** for **DDL** (Data Definition Language, e.g., CREATE TABLE).<br>

<br>

# Transaction isolation levels
In PostgreSQL, you can request any of the 4 standard transaction isolation levels, but internally **only three** distinct isolation levels are implemented.<br>
**PostgreSQL's Read uncommitted** mode behaves like **Read committed**.<br>
This is because it is the only sensible way to map the standard isolation levels to PostgreSQL's **multiversion concurrency control** architecture.

<br>

# Examples
## SET transaction isolation level for current session
### `SET`
```sql
so_rs=# BEGIN;
BEGIN

so_rs=# SET TRANSACTION ISOLATION LEVEL read uncommitted;
SET

so_rs=# SET TRANSACTION ISOLATION LEVEL read committed;
SET

so_rs=# SET TRANSACTION ISOLATION LEVEL repeatable read;
SET

so_rs=# SET TRANSACTION ISOLATION LEVEL serializable;
SET
```

<br>

### `BEGIN`
Variants:
- `BEGIN ISOLATION LEVEL serializable;`
- `BEGIN TRANSACTION ISOLATION LEVEL serializable;`

```sql
demo=# BEGIN ISOLATION LEVEL serializable;
BEGIN
Time: 3.070 ms
demo=*# SHOW transaction isolation level;
 transaction_isolation
-----------------------
 serializable
(1 row)

Time: 1.154 ms
demo=*# END;
COMMIT
Time: 1.406 ms
demo=#
```

<br>

## SHOW transaction isolation level for current session
### Current transaction isolation
```sql
so_rs=# SHOW transaction isolation level;
 transaction_isolation
-----------------------
 serializable
(1 row)
```

<br>

### Default transaction isolation
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