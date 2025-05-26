# pg_hba.conf
The `pg_hba.conf` defines **authentication policies** or **authentication rows**.<br>
Every **authentication policy** map **role** on **database**, **connection type**, **client’s remote socket**, **auth method**.<br>

By default psql uses **peer** auth method.<br>

**Each auth row has format**:<br>
`TYPE  DATABASE  USER  ADDRESS  METHOD`

- **TYPE** specifies **connection type**:
  - **local** (aka local connection): connection over **UDS** (Unix Domain Socket);
  - **host**: connection over **TCP/IP** with or without SSL;
  - **hostssl**: _host_ **with** SSL;
  - **hostnossl**: _host_ **without** SSL;
- **DATABASE**: specifies the database(s) name(s) for this record match;
- **USER**: comma-separated list of **roles** or `all` is allowed for this field as well.;
- **ADDRESS** (if relevant for the connection type): contains either a **hostname** or **IP** (in CIDR format) of client or one of special keyword:
  - **all**
  - **samehost**
  - **samenet**
- **METHOD** specifies **auth method**
  - **trust**: allow connection unconditionally;
  - **reject**: reject the connection unconditionally;
  - **md5**: challenge response mechanism over **md5**;
  - **scram-sha-256**: challenge response mechanism over **sha256**;
  - **password**: clear password;
  - **peer**:
    - obtains the provided by client username from OS and check they are match;
    - this method is only available for **local connection** types and on OS providing the `getpeereid()` function;

All three methods **password**, **md5*** and **scram-sha-256** are also called **password based auth**.<br>

_Postgresql passwords_ are **separated** from _OS user passwords_. The **password for each user** is stored in the `pg_authid` system catalog.<br>
_Postgresql passwords_ are managed with the SQL command `CREATE ROLE ...` and `ALTER ROLE ...`.<br>
**If no password has been set** for user the stored password is `NULL` and **password based auth** will **always fail** for that user.<br>

<br>

After installation is completed there is
- user `postgres` is created **by default**;
- user `postgres` **by default** is `superuser` and has **all privileges**;
- user `postgres` **by default** has **no password**;

So, by default, the user `postgres` is locked and **cannot** be used for **remote connections**.<br>

To **unlock** `postgres` account for **remote connections**:
1. Allow access from any IP:
```bash
echo "host  all  all  0.0.0.0/0  md5" | sudo tee -a /etc/postgresql/12/main/pg_hba.conf
```
2. Set password for `postgres` user:
```bash
sudo -u postgres psql -c "ALTER ROLE postgres WITH PASSWORD 'new_password';"
```

<br>

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
