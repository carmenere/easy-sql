# postgres
`postgres` launches a **new server**. `postmaster` is a **deprecated** alias of `postgres`.<br>
By default, **postgres** server starts in the foreground, **doesn't** detach from **controlling terminal** and writes its output to the `stderr` stream.<br>
Option `-l` changes this dafult behaviour.<br>

## postgres options
|Option|Description|
|:-----|:----------|
|`-D datadir`<br>`--pgdata=datadir`|Sets **data directory**.<br>If this option is **omitted**, the environment variable `PGDATA` is used.|
|`-c name=value`|Sets the **configuration parameter** supported by PostgreSQL.<br>`-c` can appear **multiple times** to set multiple parameters.|
|`-C name`|Prints the **value** of the **configuration parameter** supported by PostgreSQL.|
|`-h hostname`|Sets the `listen_addresses` **configuration parameter**.|
|`-p port`|Sets the `port` **configuration parameter**.|
|`-l`|Enables **SSL**. PostgreSQL **must** have been **compiled** with **support for SSL** for this option to be available.|
|`-V`<br>`--version`|Print the postgres **version** and exit.|

## Examples
```bash
postgres -D  /usr/local/var/postgresql@12 -c log_statement=all 1>&2
```

<br>

```bash
postgres -D /usr/local/var/postgresql@12 -c log_statement=all -c logging_collector=off
```

<br>

```bash
postgres -D /usr/local/var/postgresql@12 -c config_file=/dev/null -c log_statement=all
```