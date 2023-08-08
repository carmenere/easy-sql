# pg_ctl
`pg_ctl` is used for **initialize**, **start**, **stop**, **restart**, **reload** and **displaying the status** of PostgreSQL server.<br>

Although the PostgreSQL server can be started manually, `pg_ctl` encapsulates tasks such as **redirecting log output** and properly **detaching** from the terminal and process group.<br>

Subcommands:
- `pg_ctl initdb` invokes the `initdb` command.
- `pg_ctl start` launches a **new server** in the background. 
- `pg_ctl stop` shuts down the **server** that is running in the specified data directory.
- `pg_ctl restart` effectively executes a `stop` followed by a `start`. This allows change configuration-file options that cannot be changed without full server restart.
- `pg_ctl reload` simply sends to the **server process** a `SIGHUP` signal, causing it to **reread** its **configuration files**. This allows change configuration-file options that **don't require** full server restart.
- `pg_ctl status` checks whether a server is running in the specified data directory. If it is, the server's PID and the command line options that were used to invoke it are displayed.


<br>

## pg_ctl options
By default, **postgres** server starts in the foreground, **doesn't** detach from **controlling terminal** and writes its output to the `stderr` stream.<br>
Option `-l` changes this dafult behaviour.<br>

|Option|Description|
|:-----|:----------|
|`-D datadir`<br>`--pgdata=datadir`|Sets **data directory**.<br>If this option is **omitted**, the environment variable `PGDATA` is used.|
|`-l filename`<br>`--log=filename`|**Redirects** all server's log output to filename.|
|`-o options`<br>`--options=options`|Specifies options to be passed directly to the postgres command.<br>The options should usually be surrounded by single or double quotes.<br>`-o` can be specified **multiple times**.|

<br>

## Examples
```bash
pg_ctl -D  /usr/local/var/postgresql@12 -o '-c config_file=/dev/null' -l /tmp/pg_startup.log  start
```

<br>

```bash
pg_ctl -D  /usr/local/var/postgresql@12 -o '-c log_statement=all -c config_file=/dev/null' -l /tmp/pg_startup.log  start
```

<br>

```bash
pg_ctl -D  /usr/local/var/postgresql@12 -o '-c log_statement=all -c log_directory=/tmp -c log_filename=pg.log' -l /tmp/pg_startup.log  start
```

<br>

## Example of -l option
1.	Output **with** `-l` option:
```bash
pg_ctl -D /usr/local/var/postgresql@12 -l /tmp/pg_startup.log start
waiting for server to start.... done
server started
```

Content of `/tmp/pg_startup.log`:
```bash
cat /tmp/pg_startup.log
2021-12-06 18:14:13.920 MSK [6353] FATAL:  lock file "postmaster.pid" already exists
2021-12-06 18:14:13.920 MSK [6353] HINT:  Is another postmaster (PID 6330) running in data directory "/usr/local/var/postgresql@12"?
2021-12-06 18:14:19.662 MSK [6357] LOG:  starting PostgreSQL 12.8 on x86_64-apple-darwin20.4.0, compiled by Apple clang version 12.0.5 (clang-1205.0.22.9), 64-bit
2021-12-06 18:14:19.663 MSK [6357] LOG:  listening on IPv6 address "::1", port 5432
2021-12-06 18:14:19.663 MSK [6357] LOG:  listening on IPv4 address "127.0.0.1", port 5432
2021-12-06 18:14:19.664 MSK [6357] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
2021-12-06 18:14:19.674 MSK [6357] LOG:  redirecting log output to logging collector process
2021-12-06 18:14:19.674 MSK [6357] HINT:  Future log output will appear in directory "pg_log".
```

<br>

2.	Output **without** -l option:
```bash
pg_ctl -D /usr/local/var/postgresql@12 start
waiting for server to start....2021-12-06 18:19:15.513 MSK [6460] LOG:  starting PostgreSQL 12.8 on x86_64-apple-darwin20.4.0, compiled by Apple clang version 12.0.5 (clang-1205.0.22.9), 64-bit
2021-12-06 18:19:15.514 MSK [6460] LOG:  listening on IPv6 address "::1", port 5432
2021-12-06 18:19:15.514 MSK [6460] LOG:  listening on IPv4 address "127.0.0.1", port 5432
2021-12-06 18:19:15.514 MSK [6460] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
2021-12-06 18:19:15.525 MSK [6460] LOG:  redirecting log output to logging collector process
2021-12-06 18:19:15.525 MSK [6460] HINT:  Future log output will appear in directory "pg_log".
 done
server started
```

<br>

Entry `Future log output will appear in directory "pg_log"` means that `logging_collector` is `on` and **all logs** are saved to `pg_log` directory.
