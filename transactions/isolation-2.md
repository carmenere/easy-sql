# Table of contents
<!-- TOC -->
- [Table of contents](#table-of-contents)
- [Linearizability vs Serializability](#linearizability-vs-serializability)
- [Consistency](#consistency)
- [Isolation levels in ANSI SQL-92 standard](#isolation-levels-in-ansi-sql-92-standard)
- [A Critique of ANSI SQL Isolation Levels](#a-critique-of-ansi-sql-isolation-levels)
  - [Dirty write](#dirty-write)
  - [Dirty write vs. lost update](#dirty-write-vs-lost-update)
    - [Example](#example)
  - [Write skew vs. Read skew](#write-skew-vs-read-skew)
    - [Write skew](#write-skew)
    - [Read skew](#read-skew)
    - [Write skew: total sum example](#write-skew-total-sum-example)
    - [Write skew: at least one constraint example](#write-skew-at-least-one-constraint-example)
<!-- TOC -->

<br>

# Linearizability vs Serializability
**Linearizability**, in the context of distributed systems, is a **guarantee** about *single operations* on *single objects*.<br>
*Linearizability* for read and write operations refers to the **C** or **consistency** in the **CAP theorem**.<br>

**Serializability** is a **guarantee** about *transactions*, or *groups of one or more operations* over *one or more objects*. It **guarantees** that the execution of *multiple concurrent* (happening at the same time) *transactions* (groups of operations) over *multiple objects* is **equivalent to some serial execution** (total ordering) of the transactions.<br>
In other words **serializability** ensures that these transactions play out as if they happened **one after another**, **not all at once**, maintaining the system's **correctness**.
This doesn't mean they physically occur one by one - they can still happen all at once. But the **final result** *will be the same as if they happened sequentially*.<br>
*Serializability* refers to the **I** or **isolation**, in **ACID**.<br>

<br>

# Consistency
**Consistency model** defines **constraints** or **requirements** that **guarantees** that **data** will be **consistent** and the results of **reading**, **writing**, or **updating** will be **predictable**.<br>

One *consistency model* can be considered **stronger** than another if it requires **all constraints** of that *model* and more. In other words, a *model* with **fewer constraints** is considered a **weaker** *consistency model*.<br>

<br>

**Consistency models**:
- the **strict consistency** is the **strongest** *consistency model*;
- the **sequential consistency** is a **weaker** model than *strict consistency model*;
- the **causal consistency** is a **weaker** model than *sequential consistency model*;
- the **eventual consistency** is a **weak** *consistency model*. It defines that if **no** update takes a very long time, **all replicas** *eventually become consistent*;

<br>

# Isolation levels in ANSI SQL-92 standard
The **official papers** of the **ANSI SQL-92 standard** are:
- **ANSI X3.135-1992**;
- **ISO/IEC 9075:1992**;

<br>

The **ANSI SQL-92 standard** defines only **3 phenomena**:
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

The **ANSI SQL-92** standard:
- defines **4 levels** of transaction isolation;
- **guarantees** that execution of **concurrent** transactions at *isolation level* `SERIALIZABLE` is **serializable**;

<br>

The **isolation level** specifies the *kind of phenomena* that can occur during the execution of **concurrent** transactions, in other words which *phenomena* (**P1**, **P2**, and **P3**) are **possible** and **not possible** for a given *isolation level*:
|Isolation level|Dirty read|Non-repeatable read|Phantom read|
|:--------------|:---------|:------------------|:-----------|
|`READ UNCOMMITTED`|Possible|Possible|Possible|
|`READ COMMITTED`|**Not Possible**|Possible|Possible|
|`REPEATABLE READ`|**Not Possible**|**Not Possible**|Possible|
|`SERIALIZABLE`|**Not Possible**|**Not Possible**|**Not Possible**|

<br>

# A Critique of ANSI SQL Isolation Levels
The **A Critique of ANSI SQL Isolation Levels** paper (by *Microsoft researchers*) shows that the *SQL-92 standard*'s definition of *isolation levels* based on phenomena (*Dirty Reads*, *Non-Repeatable Reads*, *Phantoms*) are **incomplete**.<br>

<br>

The **A Critique of ANSI SQL Isolation Levels** provides useful notation:
- `[x=50]` means `x` has **initial value** `50`;
- `r2[x]` means transaction **2** **reads** a **record** `x`;
- `w1[x]` means transaction **1** **writes** a **record** `x`;
- `w2[x=2]` means transaction **1** **writes** `2` to a **record** `x`;
- `r1[P]` means transaction **1** **reads** a **set of records** *satisfying predicate* `P`;
- `w1[P]` means transaction **1** **writes** a **set of records** *satisfying predicate* `P`;
- `c1` means transaction **1** performs **commit** (`COMMIT`);
- `a1` means transaction **1** performs **abort** (`ROLLBACK`);

<br>

The **A Critique of ANSI SQL Isolation Levels** introduces:
- **P0**: **Dirty Write**;
- **P4**: **Lost Update**;
- **Read Skew**;
- **Write Skew**;

<br>

**Formal defenitions** of phenomena according the **A Critique of ANSI SQL Isolation Levels**:
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

|Isolation level|Dirty write|Dirty read|Non-repeatable read|Lost update|Read skew|Write skew|Phantom read|
|:--------------|:----------|:---------|:------------------|:----------|:--------|:---------|:-----------|
|`READ UNCOMMITTED`|**Not Possible**|Possible|Possible|Possible|Possible|Possible|Possible|
|`READ COMMITTED`|**Not Possible**|**Not Possible**|Possible|Possible|Possible|Possible|Possible|
|`REPEATABLE READ`|**Not Possible**|**Not Possible**|**Not Possible**|**Not Possible**|**Not Possible**|**Not Possible**|Possible|
|`SERIALIZABLE`|**Not Possible**|**Not Possible**|**Not Possible**|**Not Possible**|**Not Possible**|**Not Possible**|**Not Possible**|

<br>

## Dirty write
In postgreSQL, **to prevent** *dirty write* the `UPDATE`/`DELETE` in **T1** both lock `UPDATE`/`DELETE` for the **same row** in *another concurrent transaction* until the end of **T1** at **any** *isolation level*.<br>

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

## Dirty write vs. lost update
The difference between **dirty write** and **lost update** is that **uncommitted** data is **overwritten** or **committed** data is **overwritten**:
- **dirty write** is that a transaction **overwrites** (updates or deletes) the **uncommitted** data;
- **lost update** is that **two transactions** read the **same row to update** it but the **first** committed update is **overwritten** by the **second** committed update;

<br>

The **A Critique of ANSI SQL Isolation Levels** paper says that **any isolation level** must protect from **dirty write**. Basically, *dirty write* **doesn't occur with all isolation levels** in many databases. Basically, *lost update* **doesn't occur** in `SERIALIZABLE` isolation level in many databases and *lost update* is **prevented** using `SELECT FOR UPDATE`.<br>

<br>

### Example
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

The changes of `T1` have **lost** because `T2` **didn't** take into account any changes made by the transaction `T1`.<br>

<br>

## Write skew vs. Read skew
### Write skew
**Write skew** is **possible** when exists **complex multi-record condition** (**business rule**, **constraint**) *in one table* or *accross tables*. **Write skew** means **violation** of such **multi-record constraint**.<br>

Two transactions **read** *multiple rows*, **check** some **condition** (**business rule**, **constraint**) based on *multiple rows*, perform **update**, but **after** transactions **commit** their changes the **condition become invalidated**.<br>

In other words, *write skew* **can corrupt the data integrity**. It is a generalization of the *lost update* problem, but instead of overwriting the same record, transactions update **different records** that are **logically linked by a constraint**.<br>

How to prevent? Even **snapshot isolation** **cannot** detect *write skew*, only **serializable isolation** (the strictest level) or **explicit row locking** detects *write skew*.<br>

<br>

### Read skew
Like *write skew* the **read skew** is **possible** when exists **complex multi-record condition** (**business rule**, **constraint**) *in one table* or *accross tables*. But unlike *write skew* the **read skew** leads to a situation when **application has inconsistent data** because it sees **half** of another transaction's changes, but the **database remains in a consistent state**.<br>

Consider that there are several rows that are **logically linked by a constraint**: `row_a` and `row_b`. **Read skew** happens when **T1** **reads** one row (e.g. `row_a`), then **T2** **updates** only `row_a` or both `row_a`+`row_b` and then **T1** reads `row_b`, but **T2** **doesn't reread** `row_a`, i.e. it holds **previous** version of `row_a`.

<br>

**Note**: **non-repeatable read** is a form of **read skew** but for the **same one row**.<br>

How to prevent at the **Read committed** level? The answer is obvious: use a **single operator**. **The data visibility can change only between operators**.<br>

<br>

### Write skew: total sum example
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

### Write skew: at least one constraint example
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
