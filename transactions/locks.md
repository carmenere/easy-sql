# Locks
[Main article](https://www.postgresql.org/docs/current/explicit-locking.html)<br>
PostgreSQL provides **various lock modes** to **control concurrent access to data** in tables.<br>
Most PostgreSQL commands **automatically** acquire locks of appropriate modes to ensure that referenced tables are not dropped or modified in incompatible ways while the command executes.

<br>

## Table-level locks
You can acquire any of `table-level locks` explicitly with the command [`LOCK`](https://www.postgresql.org/docs/current/sql-lock.html).<br>
If two **lock modes conflict**, two **different** transactions **cannot** take them at the same time on the same object.<br>
In other words, the **first** transaction takes the lock, and the **second** gets blocked until the **first** one releases the lock (commits/rollbacks).<br>

Notice that some *lock modes* are **self-conflicting** (for example, `ROW EXCLUSIVE`), while others are **not self-conflicting** (for example, `ACCESS EXCLUSIVE`).<br>
**Non-conflicting** *lock modes* can be held concurrently by many transactions.<br>

<br>

#### Example
The commands `UPDATE`, `DELETE`, `INSERT`, `MERGE` acquire `ROW EXCLUSIVE` *lock mode* on the target table (in addition to `ACCESS SHARE` locks on any other referenced tables).<br>
In general, this *lock mode* will be acquired by **any command** that **modifies** data in a table.<br>
But `ROW EXCLUSIVE` is `table-level lock` and it **doesn't conflict with itself** because two concurrent transactions are allowed to modify the data in the same table concurrently.<br>
If two concurrent transactions modifies **the same row** then appropriate `row-level lock` is acquired.<br>

<br>

### pg_locks system view
The view `pg_locks` provides access to **information about the locks** held by active processes within the database server.<br>
`pg_locks` contains one row per active lockable object, requested lock mode, and relevant process.<br>
Thus, **the same lockable object might appear many times**, if multiple processes are holding or waiting for locks on it.<br>

#### Example
```sql
so_db=# SELECT locktype,mode,transactionid,relation::regclass AS relation,virtualxid FROM pg_locks WHERE locktype = 'relation';
 locktype |       mode       | transactionid | relation | virtualxid
----------+------------------+---------------+----------+------------
 relation | AccessShareLock  |               | pg_locks |
 relation | RowExclusiveLock |               | t        |
(2 rows)
```

<br>

## Row-level locks
**Every row** in a PostgreSQL table is also protected with a lock.<br>

<br>

`Row-level locks` are **not** displayed in `pg_locks` table. You can examine them with the `pgrowlocks` extension.<br>

If **concurrent** transactions modify **the same row**, one of them will **get blocked** on a `row-level locks`.<br>
You can also take `row-level locks` explicitly **without modifying** anything using `SELECT … FOR UPDATE` or `SELECT … FOR SHARE`, which lets you temporarily prevent changes to a set of rows.<br>

For example, the `FOR UPDATE` *lock mode* is also acquired by any `DELETE` on a row, and also by an `UPDATE` that modifies the values of certain columns.<br>
