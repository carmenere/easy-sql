# pg_hba.conf
`pg_hba.conf` defines **authentication policies**.<br>
Every **authentication policy** map **role** on **database**, **connection type**, **client’s remote socket**, **auth method**.<br>
Example:
```bash
$ cat /etc/postgresql/12/main/pg_hba.conf 
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     trust

# IPv4 local connections:
host    all             all             0.0.0.0/0               md5

# IPv6 local connections:
host    all             all             ::1/128                 trust

# Allow replication connections from localhost, by a user with the replication privilege.
local   replication     all                                     trust
host    replication     all             127.0.0.1/32            trust
host    replication     all             ::1/128                 trust
host    all             all             0.0.0.0/0               md5
```

<br>

## TYPE
`TYPE` specifies **connection** type.
`TYPE` can be:
|Value|Description|
|:----|:----------|
|**local**|Unix-domain socket|
|**host**|Either a plain or SSL-encrypted TCP/IP socket|
|**hostssl**|Either a plain or SSL-encrypted TCP/IP socket|
|**hostnossl**|Plain TCP/IP socket|

<br>

## ADDRESS       
**Allowed source** IPv4/IPv6 prefixes.

<br>

## DATABASE
Specifies the database(s) name(s) for this record match.

<br>

## USER
A comma-separated list of **roles** or `all` is allowed for this field as well.

<br>

## METHOD
Methods `md5` and `scram-sha-256` use password set by `CREATE USER` or `ALTER ROLE` command.<br>
If **no** password has been set up for a user, the stored password is `NULL` and password authentication will **always fail** for that user.<br>
This methods are supported on **all** connection types (**local**, **host**, ... ).<br>

Method `peer` (aka **peer authentication**) is for *Unix* and method `sspi` is for *Windows*.<br>
The `peer` method obtains the client's user name from the **OS**: if client's user name matches the OS username – auth is **ok**, if not – auth **fails**.<br>
**Peer authentication** is only available on OS providing the `getpeereid()` function.<br>
This method is **only** supported on connection type **local**.

After installation is completed there is user `postgres` is created. 
- user `postgres` by default is `superuser` and has **all privileges**. 
- user `postgres` by default has **no password**.

By default, the user `postgres` is locked and **cannot** be used over TCP/IP connections. To login at user `postgres` in TCP/IP connections password **must** be set for user `postgres`.
