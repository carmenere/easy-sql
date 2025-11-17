# OCC vs. PCC
**Resource contention** (**concurrent access**) is a situation when two or more processes/threads **try to access** to the **same resources simultaneously**.<br>
For example, **lock contention** occurs when multiple processes/threads are trying to **acquire the same lock simultaneously**, but **only one can succeed at a time**.<br>

The **concurrency control** is a subsystem that **ensures data consistency** under **concurrent access**.<br>
Thus **concurrency control** is an essential element for correctness in any system with **concurrent access** to the same data.<br>

To **measure contention**, we consider **two** main factors:
- **how often** the *resource* is **requested**;
- **how long** the *resource* is **held** each time;

**High frequency** or **long hold duration** both **increase contention**.<br>

<br>

So,
- **Low contention** means:
  - in app: **few** processes/threads are competing for the same resource;
  - in rdbms: transactions will **rarely** conflict;
- **High contention** means:
  - in app: **many** processes/threads are competing for the same resource;
  - in rdbms: transactions will **frequently** conflict;

<br>

There are **two** main **concurrency control methods**:
- **Pessimistic Concurrency Control** (**PCC**):
  - this approach assumes **high contention**, in other words that conflicting operations happen more **frequently**;
  - it is **lock based** approach: data is locked before access to it;
- **Optimistic Concurrency Control** (**OCC**):
  - this approach assumes **low contention**, in other words that conflicting operations are **rare**;
  - it **skips the locking** and take actions only when conflict actually occurs;
  - **reading does not block writing**, and **writing does not block reading and another writing**;
  - it uses **versioning** to **detect conflicts**;

<br>

**OCC** is a concurrency control mechanism that **allows concurrent execution without acquiring locks upfront**, instead OCC **relies on version information for each data item**:
- **at the beginning** each transaction reads a consistent snapshot of the database;
- any modification of data item **increases its version**;
- **at the the commit phase**, **each transaction checks version of data it has read**:
  - if it detects that **version** of data item was **changed** it means another transaction has **modified** this data item;
  - if **version** of data item is the **same** it means data was changed only by current transaction;
- if **conflicts are detected**, the transaction is **aborted**;

<br>

**OCC** is great in environments with many transactions that are *likely to modify* **different** data.<br>
**PCC** is great in environments with many transactions that are *likely to modify* the **same** data.<br>

<br>

When to use **PCC**:
- **high contention**;
- **when there is explicit requirement to provide absolute consistency of data**;

<br>

# SI. SSI. MVCC
**MVCC** (**MultiVersion Concurrency Control**), is a **concurrency control method** belonging to **OCC**<br>
**MVCC** is an **optimistic model** because it avoids locking and assumes conflicts are rare.<br>
**MVCC** is a technique where **each modification** of row creates a **new version** of the row, and the **old version** is marked as **dead**.<br>

<br>

**Snapshot Isolation** (**SI**) is a **transaction isolation level** where each transaction operates on a **consistent snapshot** of the database, but it **allows certain anomalies** which can **break serializability**, e.g. **write-skew**.<br>
**Serializable Snapshot Isolation** (**SSI**) is a **transaction isolation level** that **prevents all serialization anomalies** by tracking dependencies between transactions. **Performance** of *SSI* can be **worse** than *SI* due to the overhead of the extra checks.<br>

Both **SI** and **SSI** uses **MVCC** to provide snapshot isolation, in other words, **MVCC** is the **underlying mechanism** for **SI** and **SSI**.<br>

<br>

# Versions of tuples
An acronym **XID** means **transaction ID** (also abbreviated **xact**).<br>

**Note** that `BEGIN` command **doesn't** assign a **XID**. In PostgreSQL, when the first command is executed after a BEGIN command executed, a tixd is assigned by the transaction manager, and then the transaction starts.<br>

In PostgreSQL, **multiple versions** of a **row** (**tuple**) **may exist simultaneously**.<br>
**Every tuple** has a **header** that contains **special fields** (**xmin**, **xmax** and **hint bits**) which help Postgres **manage** **row versions** and **visibility**:
- `t_xmin` - the ID of transaction that **inserted** the row version;
- `t_xmin` - the ID of transaction that **deleted** the row version;
- `t_infomask` - **hint bits**, they are used to determine visible row or not in current transactoin;

<br>

How SQL commands changes **xmin** and **xmax** of tuples:
- `INSERT`:
  - **inserts new row**;
  - fills the **xmin** field by **current XID**;
  - fills the **xmax** field by **0**;
- `DELETE`:
  - **updates** the **xmax** field with **current XID** value, this row is now considered a **dead tuple**;
- `UPDATE` operates as a **combination** of a `DELETE` and an `INSERT`:
  - **first**, **updates** the **xmax** of the **current row** with **current XID** value, this row is now considered a **dead tuple**;
  - **then**, inserts **new row**:
    - fills the **xmin** field by **current XID**;
    - fills the **xmax** field by **0**;

<br>

**Example**:
```sql
CREATE EXTENSION IF NOT EXISTS pageinspect;

CREATE TABLE foo (id serial, name text);

INSERT INTO foo VALUES (1, 'a');
INSERT INTO foo VALUES (2, 'a');

SELECT lp, t_ctid, t_xmin, t_xmax FROM heap_page_items(get_raw_page('foo', 0));
 lp | t_ctid | t_xmin | t_xmax
----+--------+--------+--------
  1 | (0,1)  | 624049 |      0
  2 | (0,2)  | 624050 |      0
(2 rows)

UPDATE foo SET name = 'x' WHERE id = 1;

SELECT lp, t_ctid, t_xmin, t_xmax FROM heap_page_items(get_raw_page('foo', 0));
 lp | t_ctid | t_xmin | t_xmax
----+--------+--------+--------
  1 | (0,3)  | 624049 | 624051
  2 | (0,2)  | 624050 |      0
  3 | (0,3)  | 624051 |      0
(3 rows)
```

<br>

# CLOG and hint bits
**Every** transaction knows its **XID** and **isolation level**.<br>
To determine is a particular tuple **visible** or **not** in the current transaction, the current transaction must figure out states of transactions **t_xmin** **и t_xmax** of this tuple.<br>
The **status** of any transaction can be found out by **its ID** using **ProcArray** and **CLOG**.<br>
The **ProcArray** is a special runtime structure that contains information about all processes, if process is associated with **t_xmin** of tuple it means that this **transaction ID** (**xmin**) is still active.<br>
The **commit Log** (**CLOG**) is a **bit map** that stores the states of each transaction. *CLOG* uses **2 bits** to store state of transaction.<br>

So, **2 bits** give **4 states of transaction**:
- `TRANSACTION_STATUS_IN_PROGRESS`
- `TRANSACTION_STATUS_COMMITTED`
- `TRANSACTION_STATUS_ABORTED`
- `TRANSACTION_STATUS_SUB_COMMITTED`

<br>

The **CLOG** can be flushed to disk, so it is expensive operation to visit *CLOG*, because sometimes it can require access to disk.<br>
**Hint bit** are used **to optimize the discovering** states of **t_xmin** **и t_xmax** transactions.<br>
**Hint bits** store state of **t_xmin** **и t_xmax** transactions **directly in tuple** itself.<br>
**Hint bits** store all necessary information about status of **t_xmin** **и t_xmax** transactions, but they **can only be set by another transaction**, because original transaction **doesn't** know how it will be finished and cannot set *hint bits* untill it is completed.

<br>

There are **four hint bits** in `t_infomask` field for possible **4 situatuins**:
- `HEAP_XMIN_COMMITTED`: 0x0**1**00 = 0000000**1**00000000
- `HEAP_XMIN_INVALID`:   0x0**2**00 = 000000**1**000000000 
- `HEAP_XMAX_COMMITTED`: 0x0**4**00 = 00000**1**0000000000
- `HEAP_XMAX_INVALID`:   0x0**8**00 = 0000**1**00000000000

<br>

The `HEAP_XMIN_FROZEN` means that **both** bits `HEAP_XMIN_COMMITTED` and `HEAP_XMIN_INVALID` are **set**: `(HEAP_XMIN_COMMITTED|HEAP_XMIN_INVALID)` = 000000**11**00000000.<br>
The `HEAP_XMIN_FROZEN` means that the tuple in **unconditionally visible**, and its **xmin** and **xmax** should be ignored. Such tuples are called **frozen** and are visible to all transaction forever.<br>

<br>

For example, if **neither** of `HEAP_XMIN_COMMITTED` and `HEAP_XMIN_INVALID` bits is set, then either:
- the **creating transaction is still in progress**, and the **current transaction** can check this through `ProcArray`;
- if the **creating transaction is still in progress**, then the tuple is **not** visible to **current transaction** and it **can't** set any hint bits;
- if **creating transaction is completed**, it means the **current transaction** is the first checks these bits for tuple, in such case the **current transaction** have to visit the **Commit Log** (**CLOG**) to interprete value of **xmin**.
  - if the transaction with id=**xmin** was **committed** or **rolled-back**, then **current transaction** sets the `HEAP_XMIN_COMMITTED` or `HEAP_XMIN_INVALID` accordingly and then **future transactions don't** need to visit the **CLOG**;

<br>

# Snapshots
A **transaction snapshot** (or just **snapshot**) is used by **visibility rules** to define rows that current transaction can or cannot see.<br>

A **snapshot** can be represented by several transaction numbers: `[xmin, xmax, xip_list]`.<br>

When a **transaction starts**, it acquires a **snapshot** that includes:
- `xmin` (**snapshot's xmin**): the **lowest XID** that is **still active** when the snapshot was taken;
  - anything less than this committed or rolled back.
- `xmax` (**snapshot's xmax**):` xmax` = (the **highest XID** that had **completed** when the snapshot was taken + **1**);
- `xip_list` (**snapshot's xip_list**): a **list of XIDs** that were **active** when the snapshot was taken;
  - should be **greater** than `xmin` and **less** than `xmax`;

Please note that the definitions of `xmin` and `xmax` in a **snapshot** are **different** from those stored in the **tuple header**.<br>

<br>

For a specific row **to be visible** to a transaction, it must satisfy both of the following visibility rules:
- creation rule (**xmin**): the transaction that created the row version (**xmin**) must have been **committed before** the **current** transaction's snapshot was taken;
- deletion rule (**xmax**): the transaction that deleted the row version (**xmax**) must **not** have been **committed before** the **current** transaction's snapshot was taken;

<br>

**Visibility rules**:
1. Any transaction with an ID **less than** the **snapshot's xmin** is considered either **committed** and **visible**, or **rolled back** and **not** visible.
2. All transaction with an ID **greater than or equal** to the **snapshot's xmax** are considered **incompleted** and is therefore **invisible** to this snapshot.
3. Any transaction with an ID **greater than** the **snapshot'xmin** and **less than or equal** the **snapshot'xmax** is considered either **committed** and **visible**, or **rolled back** and **not** visible, **except** changes made by transactions from **xip list** are **not visible**.

<br>

**Example 1**: consider 2 snapshots: `100:100` and `100:104:100,102`:<br>
![pg_snapshot](/img/pg_snapshot.png)

<br>

**Example 2**:
```sql
SELECT txid_current();
 txid_current
--------------
       624045
(1 row)
```

```sql
SELECT pg_current_snapshot();
     pg_current_snapshot
-----------------------------
 624043:624046:624043,624044
(1 row)
```

<br>

Transaction **624045** can see rows for which `xmin < 624043`.
Transaction **624045** can see rows for which `624043 < xmin <= 624046` **except** for **624043,624044** because they are **still in progress** (not committed or rollbacked).

<br>

# Dead tuples vs. Frozen tuples
**Dead tuples** are tuples where `t_xmax != 0` and flag `HEAP_XMAX_COMMITTED` is **set**.<br>
When **dead tuples** gets **no longer visible** in **all snapshots** they **can be vacuumed away**.<br>
If not clean up **dead tuples**, they can lead to table bloat.<br>

**Frozen tuples** are **always visible** to **all** transactions.<br>
**Frozen tuples** are considered **older than any rows** and that are always **visible** in **all snapshots**. This is achieved by setting specific **hint bits** in the **tuple header**.<br>

The primary purpose of **freezing** is to prevent **transaction ID wraparound issue**.<br>
PostgreSQL's **transaction IDs are finite**, and if they **wrap around** without older transactions being **frozen**, it could lead to data corruption.<br>
**Freezing** effectively marks a tuple as **older** than any possible transaction ID.<br>

To **track** the **xmin** transaction as **frozen**, *both hint bits* are set: `HEAP_XMIN_COMMITTED` and `HEAP_XMIN_INVALID`.<br>

**Once frozen, a tuple's visibility doesn't need to be checked against transaction IDs**, which is simplifying visibility checks and reducing overhead.<br>

<br>

# Horizon
In PostgreSQL, the **xmin horizon** represents the **oldest XID** that is **still considered active**.<br>
The **xmin horizon** (aka **db horizon**) relates to the point in the transaction history beyond which *some dead tuples* **can be removed** or *some tuples* **can be frozen**.<br>

The **Vacuum process** is responsible for **freezing** sufficiently *old tuples* and **cleaning up** (and thus **reclaiming** disk space) sufficiently *old dead tuples*.<br>
However, **Vacuum** can only remove **dead tuples** whose `xmax` is **older** than the `xmin horizon`, in other words whose `xmax < xmin horizon`.<br>
If there are **dead tuples** whose `xmax` is **newer** than or **equal to** the `xmin horizon` (`xmax >= xmin horizon`) it means there's an **active transaction** that might **still need to see that tuple**, so VACUUM cannot remove such tuples yet.

<br>

So, if **xmin horizon** is held by a **long-running** transaction, VACUUM cannot effectively clean up *dead rows* and this can lead to:
- **table bloat**: accumulation of dead tuples, increasing table size and disk usage;
- **reduced performance**: more data to scan during queries, leading to slower performance;
- **transaction ID wraparound issue**: tuples are not frozen by VACUUM cannot;

<br>

# Visibility map
The **visibility map** can quickly show whether a page needs to be vacuumed or frozen. For this purpose, it provides **two bits** for each table page.<br>
The **first bit** is set for pages that contain **only up-to-date row versions**, in other words there are **no dead tuples** on page. Vacuum **skips** such pages because there is nothing to clean up.<br>
Besides, when a transaction tries to read a row from such a page, there is **no need to check its visibility**, so an **index-only scan can be used**.<br>
The **second bit** is set for pages that contain **only frozen row versions**. Vacuum **skips** such pages even for preventing XID wraparound.<br>
