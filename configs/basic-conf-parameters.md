# Configuration files
To get paths to all config files run:
```sql
my_db=# SELECT name, setting FROM pg_settings WHERE category = 'File Locations';
       name        |                   setting
-------------------+----------------------------------------------
 data_directory    | /usr/local/var/postgresql@12 
 config_file       | /usr/local/var/postgresql@12/postgresql.conf
 hba_file          | /usr/local/var/postgresql@12/pg_hba.conf
 external_pid_file |
 ident_file        | /usr/local/var/postgresql@12/pg_ident.conf
```

<br>

Path to only `postgresql.conf`:
```sql
my_db=# show config_file;
                 config_file
----------------------------------------------
 /usr/local/var/postgresql@12/postgresql.conf

```

<br>

Path to only **data directory**:
```sql
my_db=# show data_directory;
        data_directory
------------------------------
 /usr/local/var/postgresql@12
```

<br>

- `config_file` specifies the **main server configuration file** (`postgresql.conf`);
- `hba_file` specifies the configuration file for **host-based authentication** (`pg_hba.conf`);
- `data_directory` specifies the directory to use for **data storage**;


<br>

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

# Connection Settings
- `listen_addresses (string)`
- `port (integer)`
- `max_connections (integer)`

<br>

### listen_addresses (string)
Specifies the TCP/IP address(es) for server's **listening sockets**, the **default** value is `localhost`.<br>

The value takes the form of a **comma-separated list** of **host names** and/or **numeric IP addresses**. 
- the special entry `*` corresponds to **all available IP interfaces**;
- the entry `0.0.0.0` allows listening for **all IPv4** addresses;
- the entry `::` allows listening for **all IPv6** addresses. 
- if the list is **empty**, the server does not listen on any IP interface at all, in which case **only Unix-domain sockets** can be used to connect to it. 

<br>

Examples:
- `listen_addresses='*'`
- `listen_addresses='localhost,{ip}'`

<br>

While client authentication allows fine-grained control over who can access the server, `listen_addresses` controls which interfaces accept connection attempts.

<br>

### port (integer)
The **TCP port** the server listens on, `5432` by **default**.

<br>

# max_connections (integer)
Determines the **maximum** number of **concurrent** connections to the database server, the **default** is typically `100` connections.
