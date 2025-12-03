# initdb
`initdb` **creates** a **new** PostgreSQL **database cluster**. `initdb` can also be invoked via `pg_ctl`.<br>

Creating a database cluster consists of:
- **creating** the **directories** in which the cluster data will live;
- **generating** the **shared catalog tables** (**tables that belong to the whole cluster** rather than to any particular database);
- **creating** the `postgres`, `template1`, and `template0` **databases**;

<br>

`initdb` must be run under the user **that** will **own** the **server process**, because the server needs to have **access** to the **files** and **directories** that `initdb` creates.<br>

`initdb` initializes the database cluster's **default locale** and **character set encoding**. These can also be set separately for each database when it is created.<br>
`initdb` determines those settings for the `template` databases, which will serve as the **default** for **all** other databases.<br>
By default, `initdb` uses the **locale provider** `libc`, takes the **locale settings** from the **environment**, and determines the **encoding** from the **locale settings**.

<br>

## Set locale example
```bash
sed -i 's/^# *\(en_US.UTF-8\)/\1/' /etc/locale.gen
locale-gen
update-locale LANG=en_US.UTF-8
```

<br>

## initdb options
|Option|Description|
|:-----|:----------|
|`-D datadir`<br>`--pgdata=datadir`|Sets **data directory**.<br>If this option is **omitted**, the environment variable `PGDATA` is used.|
|`-U username`<br>`--username=username`|Sets the **user name** of the database **superuser**.<br>If this option is **omitted**, the name of the **effective** user running `initdb` is used.|
|`-W`<br>`--pwprompt`|Makes `initdb` prompt for a **password** to give the database **superuser**.|
|`--locale=locale`|This option sets the **default locale** for the database cluster.<br>If this option is **not** specified, the **locale** is **inherited** from the **environment** where `initdb` was run. |
|`--no-locale`|Equivalent to `--locale=C`.|




	
	
