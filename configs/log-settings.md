# Log subsystem settings
- `log_destination stderr`
- `logging_collector on/off`
- `log_directory (string)`
- `log_filename (string)`
- `log_statement (enum)`

<br>

### log_destination stderr
PostgreSQL supports several methods for logging server messages, including `stderr`, `csvlog` and `syslog`.<br>
The **default** is to log to `stderr` only. 

<br>

### logging_collector on/off
This parameter controls the **logging collector**, which is a **background process** that **captures** log messages sent to `stderr` and **redirects** them into **log files**.<br>
This approach is often more useful than logging to `syslog`, since some types of messages might not appear in `syslog` output.

<br>

### log_directory (string)
When `logging_collector` is **enabled**, this parameter determines the **directory** in which **log files** will be created, the **default** is `log`.<br>
It can be specified as an **absolute path**, or **relative** to the cluster **data directory**.<br>
This parameter can only be set in the `postgresql.conf` file or on the **server command line**. 

<br>

### log_filename (string)
When `logging_collector` is **enabled**, this parameter **sets** the **file names** of the created **log files**, the **default** is `postgresql-%Y-%m-%d_%H%M%S.log`.

<br>

### log_statement (enum)
When `logging_collector` is **enabled**, this parameter controls which **SQL statements** are logged.<br>
Valid values are `none`, `ddl`, `mod`, and `all` (**all statements**).

<br>

## Example
Set following parameters inside `postgresql.conf` file:
- `log_statement = 'all'`
- `logging_collector = on`
- `log_min_error_statement = error`
- `log_directory = 'pg_log'`
- `log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'`

<br>

Then:
- **system log** (**master process log**): `/usr/local/var/log/postgresql@12.log`.
- **queries log**: `/usr/local/var/postgresql@12/pg_log/postgresql-2021-11-12_112715.log`.

<br>

### Docker
To enable full logging of all queries in docker:
`command: ["postgres", "-c", "log_statement=all"]`