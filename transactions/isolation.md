# Read phenomena
**Read phenomena** is an issue that can occur at concurrent reading and/or writing to the same row in table.<br>

There are **4** read phenomena:
|Read phenomena|Meaning|
|:-------------|:------|
|**Dirty read**|A transaction sees any **uncommitted** changes made by another concurrent transaction.|
|**Non-repeatable read**|A transaction sees any **committed** changes made by another concurrent transaction, i.e., a transaction **re-reads** data it has previously read and finds that data has been **modified** by another committed concurrent transaction.|
|**Phantom read**|A transaction **re-executes** a query returning a **set of rows** that satisfy some condition and finds that the result **set of rows** has **changed** due to another committed concurrent transaction.|
|**Serialization anomaly**|When the **result** of parallel execution of transactions **depends on order** in which transactions are commited.<br>Example of **serialization anomaly** is **write skew anomaly**.|

<br>

**Phantom read** is a similar to **non-repeatable read**, but affects queries that search for **multiple** rows instead of one.<br>

<br>

# Snapshot isolation vs. Serializability
**Snapshot isolation** has been adopted by plenty database management systems, such as Oracle, MySQL, PostgreSQL, Microsoft SQL Server (2005 and later).<br>
The main reason for its adoption is that it allows **better performance** than **serializability**.<br>
In practice **snapshot isolation** is implemented within **multiversion concurrency control** (**MVCC**).<br>
A transaction executing under **snapshot isolation** appears to operate on a **personal snapshot** of the database, taken **at the start of the transaction**.<br>
When the transaction concludes, it will successfully commit only if the values updated by the transaction have not been changed externally since the snapshot was taken.<br>
Such a **write–write conflict** will cause the transaction to abort.<br>

<br>

# Transaction isolation levels
There are **4** **transaction isolation levels**.<br>
Every **isolation level** includes previous and **reduces** the particular type of **read phenomena**.<br>

<br>

|Isolation Level|Dirty Read|Nonrepeatable Read|Phantom Read|Serialization Anomaly|
|:--------------|:---------|:-----------------|:-----------|:--------------------|
|**Read uncommitted**|Possible <br>(not in PostgreSQL)|Possible|Possible|Possible|
|**Read committed**|**Not possible**|Possible|Possible|Possible|
|**Repeatable read**|**Not possible**|**Not possible**|Allowed, but not in PG|Possible|
|**Serializable**|**Not possible**|**Not possible**|**Not possible**|**Not possible**|

<br>

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