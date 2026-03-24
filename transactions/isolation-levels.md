# Table of contents
<!-- TOC -->
- [Table of contents](#table-of-contents)
- [Concurrency control](#concurrency-control)
  - [Pessimistic lock vs. Optimistic lock](#pessimistic-lock-vs-optimistic-lock)
  - [MVCC. SI. SSI](#mvcc-si-ssi)
- [The transaction isolation levels](#the-transaction-isolation-levels)
  - [Isolation levels according ANSI SQL-92](#isolation-levels-according-ansi-sql-92)
  - [A critique of ANSI SQL isolation levels](#a-critique-of-ansi-sql-isolation-levels)
  - [Dirty write vs. Lost update](#dirty-write-vs-lost-update)
    - [Dirty write](#dirty-write)
      - [Example](#example)
      - [How PostgreSQL prevents dirty write](#how-postgresql-prevents-dirty-write)
    - [Lost update](#lost-update)
  - [Write skew vs. Read skew](#write-skew-vs-read-skew)
    - [Write skew](#write-skew)
      - [Write skew: total sum example](#write-skew-total-sum-example)
      - [Write skew: at least one constraint example](#write-skew-at-least-one-constraint-example)
    - [Read skew](#read-skew)
- [Examples for PostgreSQL](#examples-for-postgresql)
  - [`END`](#end)
  - [SET transaction isolation level for current session](#set-transaction-isolation-level-for-current-session)
    - [`SET`](#set)
    - [`BEGIN`](#begin)
  - [SHOW transaction isolation level for current session](#show-transaction-isolation-level-for-current-session)
    - [Current transaction isolation](#current-transaction-isolation)
    - [Default transaction isolation](#default-transaction-isolation)
  - [SET default transaction isolation level](#set-default-transaction-isolation-level)
    - [Through postgresql.conf](#through-postgresqlconf)
    - [Through ALTER DATABASE](#through-alter-database)
- [Example: updating the same row simultaneously](#example-updating-the-same-row-simultaneously)
  - [Repeatable read](#repeatable-read)
  - [Read committed](#read-committed)
<!-- TOC -->

<br>

# Concurrency control
## Pessimistic lock vs. Optimistic lock
**Pessimistic lock** is exclusive lock until work is finished.<br>
**Optimistic lock** is a **lock free approach**, based on versioning, if transaction sees that version was changed it is immediately aborted.<br>

<br>

**Pessimistic lock** is a good when the **cost** of **locks** is **less** than the **cost** of **rollbacks**.<br>
It is good in the environment with **high contention for data**, i.e. when multiple concurrent transaction changes the same data.<br>

**Optimistic lock** is a good when the **cost** of **rollbacks** is **less** than the **cost** of **rolling back**.<br>
It is good in the environment with **low contention for data**.<br>

<br>

## MVCC. SI. SSI
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

# The transaction isolation levels
**Concurrent execution** of multiple _correct_ **transactions** can lead to **several types of problems** (aka **concurrent phenomena**, **concurrent anomalies**, **read phenomena** or just **phenomena**) that **violate** the _data consistency_. In other words, **concurrent anomalies** are issues that can occur at **concurrent reading** and **writing** to the **same** data item in db.<br>

<br>

## Isolation levels according ANSI SQL-92
The **official papers** of the **ANSI SQL-92 standard** are:
- **ANSI X3.135-1992**;
- **ISO/IEC 9075:1992**;

<br>

The **ANSI SQL-92** standard:
- defines **only 3 phenomena**:
  - **P1**: **dirty read**;
  - **P2**: **non-repeatable read**;
  - **P3**: **phantom** (**note**, due to *ANSI SQL-92 standard* it is called just *phantom*, **not** *phantom read*);
  - **serialization anomalies**:
    - **all other possible anomalies** are also known as **serialization anomalies**;
    - **serialization anomalies** arise when the **result** of execution of concurrent transactions **depends** on **order** in which transactions are commited;
- defines **4 levels of transaction isolation**:
  - **read uncommitted**;
  - **read committed**;
  - **repeatable read**;
  - **serializable**;
    - the **serializable level** is the **strongest** isolation level: it **fixes all serialization anomalies**;

<br>

The **isolation level** specifies which *phenomena* (**P1**, **P2**, and **P3**) are **possible** and **not possible** for a given *isolation level*:
|Isolation level|Dirty read|Non-repeatable read|Phantom read|
|:--------------|:---------|:------------------|:-----------|
|`READ UNCOMMITTED`|Possible, but **not in PG**|Possible|Possible|
|`READ COMMITTED`|**Not Possible**|Possible|Possible|
|`REPEATABLE READ`|**Not Possible**|**Not Possible**|Possible, but **not in PG**|
|`SERIALIZABLE`|**Not Possible**|**Not Possible**|**Not Possible**|

<br>

## A critique of ANSI SQL isolation levels
The **A critique of ANSI SQL isolation levels** paper (by *Microsoft researchers*) shows that the *SQL-92 standard*'s definition of *isolation levels* based on phenomena (*Dirty Reads*, *Non-Repeatable Reads*, *Phantoms*) are **incomplete**.<br>

<br>

The *A critique of ANSI SQL isolation levels* **introduces**:
- **P0**: **dirty write**;
- **P4**: **lost update**;
- **Read skew**;
- **Write skew**;

<br>

So, in conjuction with the **ANSI SQL-92** standard we have **at least 7** anomalies (below `T1` means transaction 1, `T2` means concurrent transaction 2):
- **P0**: **dirty write**;
  - the *dirty write* anomaly occurs when one transaction **overwrites UNcommitted changes** made by another transaction;
  - **fixed** by `READ UNCOMMITTED`;
- **P1**: **dirty read**;
  - the *dirty read* anomaly occurs when one transaction **sees UNcommitted changes** made by another transaction;
  - **fixed** by `READ COMMITTED`;
  - but **in PostgreSQL** it is also **fixed** by `READ UNCOMMITTED`;
- **P2**: **non-repeatable read**;
  - the *non-repeatable read* anomaly occurs when the `T1` **reads** _the same row_ **twice**, 
  whereas the `T2` **updates** (or **deletes**) this row between these reads and **commits** the change;
    - as a result, the `T1` **gets different results**;
  - **fixed** by `REPEATABLE READ`;
- **P3**: **phantom** (**note**, due to *ANSI SQL-92 standard* it is called just *phantom*, **not** *phantom read*);
  - the *phantom read* anomaly occurs when the `T1` **fetches** a **set of rows** that **satisfy** a particular **search condition**,
  while `T2` **inserts** or **deletes** some other rows satisfying that condition and **commits** the changes;
    - as a result, the `T1` **gets two different sets of rows**;
  - **fixed** by `SERIALIZABLE`;
  - but **in PostgreSQL** it is also **fixed** by `REPEATABLE READ`;
- **P4**: **lost update**;
  - the *lost update* anomaly occurs when one transaction **overwrites committed changes** made by another transaction;
  - **fixed** by `SERIALIZABLE`;
- **Read skew**;
  - *read skew* anomaly occurs when 1) exists **complex multi-record condition** (**business rule**, **constraint**) *in one table* or *accross tables* and 2) such **multi-record condition** is **violated** *at application level*, but *at database level* this constraint is **not violated**;
  - in other words, application has **inconsistent view of multiple records**;
  - **fixed** by `SERIALIZABLE`;
- **Write skew**;
  - *write skew* anomaly occurs when 1) exists **complex multi-record condition** (**business rule**, **constraint**) *in one table* or *accross tables* and 2) such **multi-record condition** is **violated** *at database level*;
  - **fixed** by `SERIALIZABLE`;

<br>

**Scenarios** for **P1**, **P2** and **P3**:
- **P1**: **dirty read**:
  - **T1** **modifies** a row with id `10`;
  - **T2** then **reads** that row with id `10` **before** **T1** performs a `COMMIT`;
  - if **T1** then performs a `ROLLBACK`, **T2** will have read a row that was **never committed** and that may thus be considered to have **never existed**;
- **P2**: **non-repeatable read**:
  - **T1** **reads** a row with id `10`;
  - **T2** then **modifies** or **deletes** that row with id `10` and performs a `COMMIT`;
  - If **T1** then attempts to **reread** the row with id `10`, it may **receive** the **modified value** or **discover** that the **row has been deleted**;
- **P3**: **phantom** (**note**, due to *ANSI SQL-92 standard* it is called just *phantom*, **not** *phantom read*):
  - **T1** reads the **set of rows** that satisfy some `<T1 search condition>`;
  - **T2** then executes SQL-statements that **generates or deletes one or more** rows that **satisfy** the `<T1 search condition>`;
  - **T1** **rereads** the **set of rows** that satisfy some `<T1 search condition>` and sees a **different** collection of rows;

<br>

The *A critique of ANSI SQL isolation levels* has provided a useful **notation**:
- `[x=50]` means `x` has **initial value** `50`;
- `r2[x]` means transaction **2 reads** a **record** `x`;
- `w1[x]` means transaction **1 writes** a **record** `x`;
- `w2[x=2]` means transaction **1 writes** `2` to a **record** `x`;
- `r1[P]` means transaction **1 reads** a **set of records** *satisfying predicate* `P`;
- `w1[P]` means transaction **1 writes** a **set of records** *satisfying predicate* `P`;
- `c1` means transaction **1** performs **commit** (`COMMIT`);
- `a1` means transaction **1** performs **abort** (`ROLLBACK`);

<br>

**Formal defenitions** of **7** anomalies:
- **Dirty write**: `w1[x]...w2[x]...((c1 or a1) and (c2 or a2) in any order)`;
- **Dirty read**: `w1[x]...r2[x]...(a1 and c2 in either order)`;
  - problem: we want to transfer `40` from `x` to `y`:
    - `[x=50, y=50]...r1[x=50]...w1[x=10]...r2[x=10]...r2[y=50]...c2...r1[y=50]...w1[y=90]...c1`
  - solutuon: we must not read in `T2` uncommitted data;
- **Non-repeatable read**: `r1[x]...w2[x]...c2...r1[x]...c1`;
  - problem: we want to transfer `40` from `x` to `y`:
    - `[x=50, y=50]...r1[x=50]...r2[x=50]...w2[x=10]...r2[y=50]...w2[y=90]...c2...r1[y=90]...c1`
- **Phantom**: `r1[P]...w2[y in P]...c2...r1[P]...c1`;
- **Lost Update**: `r1[x=1]...w2[x=10]...c2...w1[x=1+1]...c1`
- **Read skew**: `r1[x]...w2[x]...w2[y]...c2...r1[y]...(c1 or a1)`;
  - problem: we want to transfer `40` from `x` to `y`:
    - `[x=50, y=50]...r1[x=50]...w2[x=10]...w2[y=90]...c2...r1[y=90]...(c1 or a1)`
- **Write skew**: `r1[x]...r2[y]...w1[y]...w2[x]...(c1 and c2 occur)`;

<br>

Mapping between *isolation levels* and *anomalies*:
|Isolation level|Dirty write|Dirty read|Non-repeatable read|Lost update|Read skew|Write skew|Phantom read|
|:--------------|:----------|:---------|:------------------|:----------|:--------|:---------|:-----------|
|`READ UNCOMMITTED`|**Not Possible**|Possible, but **not in PG**|Possible|Possible|Possible|Possible|Possible|
|`READ COMMITTED`|**Not Possible**|**Not Possible**|Possible|Possible|Possible|Possible|Possible|
|`REPEATABLE READ`|**Not Possible**|**Not Possible**|**Not Possible**|**Not Possible**|**Not Possible**|**Not Possible**|Possible, but **not in PG**|
|`SERIALIZABLE`|**Not Possible**|**Not Possible**|**Not Possible**|**Not Possible**|**Not Possible**|**Not Possible**|**Not Possible**|

Every **upper** *isolation level* **fixes** *particular concurrent anomaly* (*phenomenon*) and **includes all** previous levels.<br>
Next level gives **more** _consistency_ but **decreases** _performance_.<br>
Why there are several levels? To choose the right **balance** between data **consistency** and **perfomance**.<br>

<br>

**Note**:
- in PostgreSQL **dirty reads** are **not allowed at all**:
  - in PostgreSQL `READ UNCOMMITTED` is **equivalent** for `READ COMMITTED`;
  - you **can** technically set the transaction isolation level to `READ UNCOMMITTED`, **but internally it is** `READ COMMITTED`;
- also PostgreSQL's `REPEATABLE READ` implementation **does not allow Phantom reads**;

<br>

When `REPEATABLE READ` or **serializable** are used the DB can throw a **serialization error**:
- `ERROR: could not serialize access due to concurrent update` for `REPEATABLE READ`;
- `ERROR: could not serialize access due to read/write dependencies among transactions` for `SERIALIZABLE`;

<br>

So, **application** must be ready to handle a **serialization error**, in other words it must be able to **retry transactions** that have been completed with a serialization failure.<br>

<br>

## Dirty write vs. Lost update
The difference between **dirty write** and **lost update** is that **uncommitted** data is **overwritten** or **committed** data is **overwritten**:
- **dirty write** is that a transaction **overwrites** the **uncommitted** data;
- **lost update** is that **two transactions** read the **same row to update** it but the **first** committed update is **overwritten** by the **second** committed update;

<br>

### Dirty write
The **dirty write** anomaly occurs when a transaction **overwrites uncommitted changes** made by another transaction.<br>

The *A critique of ANSI SQL isolation levels* paper says that **any isolation level** must protect from **dirty write**. Basically, *dirty write* **doesn't occur at all isolation levels** in many databases.<br>

<br>

#### Example
Table:
|id|name|stock|
|:-|:---|:----|
|1|Apple|10|
|2|Orange|20|

<br>

These steps below shows **dirty write**:
|Steps|T1|T2|Explanation|
|:----|:-|:-|:----------|
|Step 1|`BEGIN;`|||
|Step 2||`BEGIN;`||
|Step 3|`SELECT stock FROM product WHERE id = 2;`<br>20||**T1** reads `20`|
|Step 4||`SELECT stock FROM product WHERE id = 2;`<br>20|**T2** reads `20`|
|Step 5|`UPDATE product SET stock = '13' WHERE id = 2;`||**T1** **updates** `20` to `13`|
|Step 6||`UPDATE product SET stock = '16' WHERE id = 2;`|**T2** **updates** `13` to `16` **before** **T1** commits, **dirty write occurs**|
|Step 7|`COMMIT;`|||
|Step 8||`COMMIT;`||

<br>

#### How PostgreSQL prevents dirty write
In PostgreSQL, **to prevent** *dirty write* the `UPDATE`/`DELETE` in **T1** both lock `UPDATE`/`DELETE` for the **same row** in *another concurrent transaction* until the end of **T1** at **any** *isolation level*.<br>

|Steps|T1|T2|Explanation|
|:----|:-|:-|:----------|
|Step 1|`BEGIN;`|||
|Step 2||`BEGIN;`||
|Step 3|`UPDATE foo SET x=10 WHERE id = 1;`|||
|Step 4||`UPDATE foo SET x=20 WHERE id = 1`|**T2** **cannot** update `x` to `20` **before** **T1** **commits**/**abort** because this row with `id=1` is **locked** by **T1**|
|Step 5|`COMMIT;`|||
|Step 6||`UPDATE foo SET x=20 WHERE id = 1`|**T2** continues execution|

<br>

The result of **Step 6** depends on *isolation level*:
- for `READ COMMITTED` **T2** can update this row;
- **but** for `REPEATABLE READ` and `SERIALIZABLE` - **cannot** and db returns **error**;

<br>

### Lost update
The **lost update** anomaly occurs when one transaction **overwrites committed changes** made by another transaction, i.e. changes of `T1` have **lost** because `T2` **didn't take into account** any changes made by the transaction `T1` and **overwritten** them all.<br>

Basically, *lost update* **doesn't occur** in `SERIALIZABLE` isolation level in many databases and *lost update* is also **prevented** using `SELECT FOR UPDATE`.<br>

<br>

These steps below shows **lost update**:
|Steps|T1|T2|Explanation|
|:----|:-|:-|:----------|
|Step 1|`BEGIN;`|||
|Step 2||`BEGIN;`||
|Step 3|`SELECT stock FROM product WHERE id = 2;`<br>20||**T1** reads `20`|
|Step 4||`SELECT stock FROM product WHERE id = 2;`<br>20|**T2** reads `20`|
|Step 5|`UPDATE product SET stock = '13' WHERE id = 2;`||**T1** **updates** `20` to `13`|
|Step 6|`COMMIT;`|||
|Step 7||`UPDATE product SET stock = '16' WHERE id = 2;`|**T2** **updates** `13` to `16` ****after**** **T1** committed|
|Step 8||`COMMIT;`|**T2** commits: **lost update occurs**|

<br>

## Write skew vs. Read skew
### Write skew
**Write skew** anomaly occurs when 1) exists **complex multi-record condition** (**business rule**, **constraint**) *in one table* or *accross tables* and 2) such **multi-record condition** is **violated** *at database level*.<br>

<br>

Two concurrent transactions:
1) *read* **logically linked** *rows* (from **one** table or from **several** tables);
2) **check** some **multi-record condition** based on these **logically linked** *rows*;
3) then each transaction **updates disjoint** subsets of **logically linked** *rows*;
4) **after** each transaction **commits** the **multi-record condition** becomes **violated**;

<br>

In other words, *write skew* **can corrupt the data** (**violate integrity**). It is a generalization of the *lost update* problem, but instead of overwriting the same record, transactions update **different records** that are **logically linked by a constraint**.<br>

How to prevent? Even **snapshot isolation** **cannot** detect *write skew*, only **serializable isolation** (the strictest level) or **explicit row locking** detects *write skew*.<br>

<br>

#### Write skew: total sum example
Consider table:
|id|amount|
|:-|:-|
|10|3|
|11|3|

<br>

Consider **business rule**: the **total sum** of values in *column* `amount` for rows with `id=10` and `id=11` **cannot** exceed `10`.<br>

<br>

**Write skew** anomaly happens when two concurrent transactions read the **old rows** with `id=10` and `id=11` and **independently decide** that their updates for column `amount` will **not** violate the *constraint*: **T1** perform `amount = amount + 6` for row with `id=10` and **T2** perform `amount = amount + 6` for row with `id=11`. So, we get:
|id|amount|
|:-|:-|
|10|9|
|11|9|

**As a result**, *column* `amount = 18` for rows with `id=10` and `id=11` and it is **write skew**.<br>

<br>

#### Write skew: at least one constraint example
Consider table of `doctors`:
|name|on_duty|
|:-|:-|
|Alice|true|
|Bob|true|

<br>

Consider **business rule**: **at least one** of doctor can be *on call* (*on duty*).<br>

<br>

**Write skew** anomaly happens when two concurrent transactions read the **old rows** with `name=Alice` and `name=Bob` and **independently decide** that their updates for column `on_duty` will **not** violate the *constraint* because `SELECT COUNT(*) FROM doctors WHERE on_call = true` returned `2` doctors that are *on duty*: **T1** perform `true = false` for row with `name=Alice` and **T2** perform `true = false` for row with `name=Bob`.<br>

<br>

- **T1**:
```sql
BEGIN TRANSACTION;
DO $$ BEGIN
  IF (SELECT COUNT(*) FROM doctors WHERE on_call = true) >= 1 THEN
    UPDATE doctors SET on_call = false WHERE name = 'Alice';
  END IF;
END $$;
COMMIT; 
```
- **T2**:
```sql
BEGIN TRANSACTION;
DO $$ BEGIN
  IF (SELECT COUNT(*) FROM doctors WHERE on_call = true) >= 1 THEN
    UPDATE doctors SET on_call = false WHERE name = 'Bob';
  END IF;
END $$;
COMMIT; 
```

**As a result**, **after** both transaction has **commited** there are will **no doctor on duty** and **constraint will be violated**:
```sql
example=# select * from doctors ;
 name  | on_call
-------+---------
 Bob   | f
 Alice | f
(2 rows)
```

<br>

### Read skew
**Read skew** anomaly occurs when 1) exists **complex multi-record condition** (**business rule**, **constraint**) *in one table* or *accross tables* and 2) such **multi-record condition** is **violated** *at application level*, but *at database level* this constraint is **not violated**. In other words, application has **inconsistent view of multiple records**.<br>

Consider that there are several rows that are **logically linked by a constraint**: `row_a` and `row_b`. **Read skew** happens when **T1** **reads** one row (e.g. `row_a`), then **T2** **updates** both `row_a` and `row_b`, then **T1** reads `row_b`, but **T2** **doesn't reread** `row_a`, i.e. it holds **previous** version of `row_a`.<br>

<br>

**Note**: **non-repeatable read** is a form of **read skew** but for the **same one row**.<br>

How to prevent at the **Read committed** level? The answer is obvious: use a **single operator**. **The data visibility can change only between operators**.<br>

<br>

# Examples for PostgreSQL
In PostgreSQL, you can request any of the 4 standard transaction isolation levels, but internally **only three** distinct isolation levels are implemented.<br>
In PostgreSQL `READ UNCOMMITTED` is **equivalent** for `READ COMMITTED`. This is because it is the only sensible way to map the standard isolation levels to PostgreSQL's **multiversion concurrency control** architecture.<br>

<br>

## `END`
PostgreSQL has special command `END` that **calls appropriate command**: `COMMIT` or `ROLLBACK`.<br>

<br>

## SET transaction isolation level for current session
### `SET`
```sql
BEGIN;
BEGIN

SET TRANSACTION ISOLATION LEVEL read uncommitted;
SET

SET TRANSACTION ISOLATION LEVEL read committed;
SET

SET TRANSACTION ISOLATION LEVEL repeatable read;
SET

SET TRANSACTION ISOLATION LEVEL serializable;
SET
```

<br>

### `BEGIN`
Variants:
- `BEGIN ISOLATION LEVEL serializable;`
- `BEGIN TRANSACTION ISOLATION LEVEL serializable;`

```sql
BEGIN ISOLATION LEVEL serializable;

SHOW transaction isolation level;
 transaction_isolation
-----------------------
 serializable
(1 row)

END;
COMMIT
```

<br>

## SHOW transaction isolation level for current session
### Current transaction isolation
```sql
SHOW transaction isolation level;
 transaction_isolation
-----------------------
 serializable
(1 row)
```

<br>

### Default transaction isolation
```sql
SHOW default_transaction_isolation;
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
ALTER DATABASE so_rs SET DEFAULT_TRANSACTION_ISOLATION TO 'serializable';
ALTER DATABASE

SHOW default_transaction_isolation;
 default_transaction_isolation
-------------------------------
 serializable
(1 row)
```

<br>

# Example: updating the same row simultaneously
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
