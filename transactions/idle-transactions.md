# Find and show stuck transactions
- `xact_start` can be `null`
```sql
SELECT pid, usename, state, query_start, query  
FROM pg_stat_activity
WHERE (state = 'idle in transaction');
```
<br>

- `xact_start` is not `null`
```sql
SELECT pid, usename, state, query_start, query  
FROM pg_stat_activity
WHERE (state = 'idle in transaction') and xact_start is not null;
```

<br>

### idle_in_transaction_session_timeout (integer)
Terminate any session that has been idle (that is, waiting for a **client** query) **within** an open **transaction** for longer than the specified amount of **milliseconds** (another **unit** can be specified, i.e., `min`).<br>
By default, `idle_in_transaction_session_timeout = 0`, a **zero** value **disables** the timeout.<br>


<br>

### idle_session_timeout (integer)
Terminate any session that has been idle (that is, waiting for a **client** query), but **not** within an open transaction, for longer than the specified amount of **milliseconds** (another **unit** can be specified, i.e., `min`).<br>
By default, `idle_in_transaction_session_timeout = 0`, a **zero** value **disables** the timeout.<br>

<br>

These settings can be set at the **current session level** with commands like `SET`, or can be set as **defaults** to a **database** with `ALTER DATABASE`, or to a **user** with `ALTER USER`, or to the entire instance through `postgresql.conf`.

<br>

Examples:
```sql
SET SESSION idle_in_transaction_session_timeout = '5min';
SET SESSION idle_in_transaction_session_timeout = '300000';
```