# SET transaction isolation level for current session
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

# SHOW transaction isolation level for current session
## CURRENT
```sql
so_rs=# SHOW transaction isolation level;
 transaction_isolation
-----------------------
 serializable
(1 row)
```

<br>

## DEFAULT
```sql
so_rs=# SHOW default_transaction_isolation;
 default_transaction_isolation
-------------------------------
 read committed
(1 row)
```

<br>

# SET default transaction isolation level
## Through postgresql.conf
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

## Through ALTER DATABASE
```sql
so_rs=# ALTER DATABASE so_rs SET DEFAULT_TRANSACTION_ISOLATION TO 'serializable';
ALTER DATABASE

so_rs=# SHOW default_transaction_isolation;
 default_transaction_isolation
-------------------------------
 serializable
(1 row)
```