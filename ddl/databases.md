# Template databases
The **template database** (aka **source databases**) is a database which will be used to create **new database** in `CREATE DATABASE` command.<br>
The `template1` is the **default** *source database* that is used by `CREATE DATABASE`.<br>

The `template0` should never be modified. The `template0` is a **clean template** that **does not** contain user objects and is **suitable** for creating databases with **different encodings** or **locales**.<br>

But you can **add objects** to `template1`, which **by default** will be copied into databases created later. For example, the `template1` may contain **encoding-specific** or **locale-specific** settings.<br>

<br>

# `CREATE DATABASE`
```sql
CREATE DATABASE db_name
WITH
   [OWNER [=] role_name]
   [TEMPLATE [=] template]
   [ENCODING [=] encoding]
   [LOCALE [=] locale ]
   [LOCALE_PROVIDER [=] locale_provider ]
   [BUILTIN_LOCALE [=] builtin_locale ]
   [ICU_LOCALE [=] icu_locale ]
   [LC_COLLATE [=] collate]
   [LC_CTYPE [=] ctype]
   [TABLESPACE [=] tablespace_name]
   [ALLOW_CONNECTIONS [=] true | false]
   [CONNECTION LIMIT [=] max_concurrent_connection];
```

<br>

**Parameters**:
- `OWNER`
  - assigns a role that will be the **owner** of the *new database*;
  - the **default** is the role you use to execute the `CREATE DATABASE` statement;
- `TABLESPACE`
  - sets the **tablespace name** for the *new database*;
  - the **default** is the **tablespace** of the **template database**;
- `CONNECTION LIMIT`
  - sets the **maximum concurrent connections** to the *new database*;
  - the **default** is `-1` which means **unlimited**;
- `ALLOW_CONNECTIONS`
  - the `ALLOW_CONNECTIONS` parameter is a **boolean** value. If it is `false`, you **cannot connect** to the database;
- `TEMPLATE`
  - sets the **template database** which will be used to create *new database*;
  - **by default** PostgreSQL uses the `template1` database as the **default template database** if you don’t explicitly specify the *template database*;
- `ENCODING`
  - sets the **character set** (aka **encoding** in pg) for the *new database*;
- `LC_COLLATE`
  - sets the **collation** (`LC_COLLATE`) that the *new database* will use;
  - the **default** is the `LC_COLLATE` of the **template database**;
  - this parameter affects the **sort order** of strings in queries that contain the `ORDER BY` clause;
- `LC_CTYPE`
  - sets the **character classification** that the *new database* will use;
  - the **default** is the `LC_CTYPE` of the **template database**;
- `LOCALE` 
  - sets the default **collation** and **character classification** in the new database;
  - can be overridden by setting `lc_collate`, `lc_ctype`, `builtin_locale`, or `icu_locale` individually;
  - if `LOCALE_PROVIDER` is `builtin`, then `LOCALE` or `BUILTIN_LOCALE` must be specified and set to either `C`, `C.UTF-8`, or `PG_UNICODE_FAST`;
- `LOCALE_PROVIDER`
  - sets the **provider** to use for the default **collation** in this database;
  - possible values are `builtin`, `libc` or `icu` (if the server was built with ICU support);
- `ICU_LOCALE` 
  - sets the **ICU locale**;
- `BUILTIN_LOCALE`
  - sets the **builtin provider locale** for the database default **collation** and **character classification**, overriding the setting `LOCALE`;
  - еhe **locale provider** must be `builtin`;

https://www.postgresql.org/docs/current/sql-createdatabase.html


https://www.postgresql.org/docs/current/app-initdb.html
https://www.postgresql.org/docs/current/multibyte.html
https://www.postgresql.org/docs/current/locale.html
https://www.postgresql.org/docs/current/manage-ag-templatedbs.html
