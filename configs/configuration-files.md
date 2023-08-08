# Configuration files
**Data directory** (aka **PGDATA**) is a directory where cluster’s data will be stored.<br>

There are 2 config files:
- `postgresql.conf` is the **main configuration file**;
- `pg_hba.conf` defines **authentication policies**;

<br>

To **get** paths to **data directory** and **all config files** run:
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

To **get** path to `postgresql.conf`:
```sql
my_db=# show config_file;
                 config_file
----------------------------------------------
 /usr/local/var/postgresql@12/postgresql.conf

```

<br>

To **get** path to **data directory**:
```sql
my_db=# show data_directory;
        data_directory
------------------------------
 /usr/local/var/postgresql@12
```
