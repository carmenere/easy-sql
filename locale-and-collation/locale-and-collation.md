# Table of contents
<!-- TOC -->
- [Table of contents](#table-of-contents)
- [Encoding](#encoding)
  - [Server encoding](#server-encoding)
  - [Client encoding](#client-encoding)
- [Locale](#locale)
  - [Configure encoding and locale using `initdb`](#configure-encoding-and-locale-using-initdb)
  - [Configure encoding and locale using `CREATE DATABASE`](#configure-encoding-and-locale-using-create-database)
- [Collation in pg](#collation-in-pg)
  - [Examples](#examples)
<!-- TOC -->

<br>

# Encoding
**Encoding** is the **process of converting** human-readable characters (e.g., `A` `é`) into a **binary format**.<br>
**Encoding** defines the rules on how alphanumeric characters are represented in **binary format**.<br>

The **encoding** you choose significantly affects your database in two ways:
- it impacts **data compatibility**;
- it impacts **storage space**;
  - for example, `utf8` can use **up to 4 bytes per character**, while `latin1` uses **just 1 byte per character** for *Western European languages*;

Using the **wrong** encoding can lead to **data corruption**. For example, a *limited encoding* like `ASCII` **only** supports basic English characters, if you try to store **characters outside this range**, such as accents or emojis, you may encounter **errors**.<br>

<br>

There 2 places of encoding:
- **database encoding**:
- **client encoding**:

<br>

## Server encoding
PostgreSQL supports a variety of different **character sets**, also known as **encoding**: [PostgreSQL encodings](https://www.postgresql.org/docs/current/multibyte.html).<br>
The **encoding** determines **how text data is stored** and **interpreted** in a concrete database.<br>

You can check your **server's encoding** with the command:
```sql
SHOW server_encoding;
```

<br>

The **default encoding** is selected **during initialization** database cluster using `initdb`.<br>
It can **only** be overridden when you **create new** database in `CREATE DATABASE db_name`, so you can have **multiple databases each with a different character set**.<br>
**By default** `CREATE DATABASE db_name` uses `template1`, which **inherits encoding** that was set **during initialization** using `initdb`.<br>
The **encoding** of the *new database* must be compatible with the **encoding** of the *template database*, **unless** `template0` is used as the template.<br>
The `template0` is a **clean template** that **doesn't** contain any user objects and is **suitable** for creating database with **any custom encoding** or **locale**.<br>

<br>

## Client encoding
**Clients** connecting to a PostgreSQL server **can specify** their **own character encoding**, PostgreSQL handles **automatic character set conversion** between the *client's encoding* and the *server's database encoding*.<br>

<br>

You can check your **client's encoding** with the command:
```sql
SHOW client_encoding;
```

<br>

PostgreSQL provides mechanisms for converting between different character sets. You can create a **new conversion** using the SQL command `CREATE CONVERSION`.<br>

**Example**: create a conversion from `UTF8` to `LATIN1` using a **custom-made** `myfunc1` function:
```sql
CREATE CONVERSION myconv FOR 'UTF8' TO 'LATIN1' FROM myfunc1;
```

<br>

Also PostgreSQL supports **automatic character set conversion** between the *server* and connecting *client* **for certain character set combinations** as described in the `pg_conversion` system catalog.<br>
To **enable** *automatic character set conversion*, you have to tell PostgreSQL server the **character set** (**encoding**) you would like to use in the **client**. There are several ways to accomplish this:
- using the `\encoding` command in `psql`;
  - `\encoding` allows you to change client encoding on the fly;
  - for example, to change the encoding to `SJIS`:
    - `\encoding SJIS`
- using this SQL command:
```sql
SET CLIENT_ENCODING TO 'value';
```

<br>

To **return** to the **default client's encoding**:
```sql
RESET client_encoding;
```

<br>

# Locale
The **locale categories** `LC_COLLATE` and `LC_CTYPE` are **determined when** a database is **created**, and **cannot** be changed **after** database was created.<br>
**Other** locale settings, e.g. `LC_MESSAGES` and `LC_MONETARY` are **initially determined** by the **environment** the server is started in, but **can be changed on-the-fly**.<br>
You can check the locale settings using the `SHOW` command:
- `SHOW LC_COLLATE;`
- `SHOW LC_CTYPE;`
- `SHOW LC_MESSAGES;`
- `SHOW LC_TIME;`
- ...

You can use **different settings** for **different databases**, but **once** a database is **created**, you **cannot** change them for that database **anymore**.<br>

The other locale categories **can** be changed whenever desired by setting the server configuration parameters that have the same name as the locale categories.<br>
The **default values** for these categories are determined when `initdb` is run, and those values are used when **new databases** are created, **unless** specified otherwise in the `CREATE DATABASE` command.<br>

<br>

**Encoding** and **locale categories** `LC_COLLATE` and `LC_CTYPE` **impacts** on the following SQL features:
- **sorting**: influences the **sort order** in `ORDER BY` clause for **textual data**;
- **pattern matching**: affects the behaviour of `LIKE` clause and `~` operator;
- **functions**: impacts functions like `upper()`, `lower()`;
- **indexes**: for example, the ability to use **indexes** with `LIKE` clause;

<br>

A **locale provider** specifies **which library** defines the locale **behavior** for **collations** and **character classifications**.<br>
PostgreSQL provides **3** primary providers:
- `builtin`
  - the `builtin` **provider** uses **built-in operations**;
  - **only** the `C`, `C.UTF-8`, and `PG_UNICODE_FAST` locales are supported for this provider;
    - the `C` **locale** behavior is **identical** to the `C` **locale** in the `libc` **provider**, when using this locale, the behavior may depend on the database encoding;
    - the `C.UTF-8` **locale** is available only for when the database **encoding** is `UTF-8`, and the behavior is based on **Unicode**
      - the **collation** uses the code point values only;
      - The **regular expression** character classes are based on the **POSIX Compatible** semantics, and the case mapping is the **simple** variant;
    - the `PG_UNICODE_FAST` **locale** is available only when the database **encoding** is `UTF-8`, and the behavior is based on **Unicode**;
      - the **collation** uses the code point values only;
      - the **regular expression** character classes are based on the "Standard" semantics, and the case mapping is the **full** variant;
- `libc`
  - the `libc` **provider** uses the **operating system's** `C` **library**;
- `icu`
  - the `icu` **provider** uses the **external** `ICU` **library**;

<br>

**Example**:
- initialize a database cluster using the **ICU provider**: `initdb --locale-provider=icu --icu-locale=en`;

<br>

## Configure encoding and locale using `initdb`
initdb creates a new PostgreSQL database cluster.

By default, `initdb` uses the **locale provider** `libc`, takes the **locale settings** from the **environment**, and determines the **encoding** from the **locale settings**.

Creating a database cluster consists of creating the directories in which the cluster data will live, generating the shared catalog tables (tables that belong to the whole cluster rather than to any particular database), and creating the postgres, template1, and template0 databases.

`initdb` initializes the database cluster's **default locale** and **character set encoding**.
**By default**, `initdb` uses the locale provider `libc`.

To choose a different locale for the cluster, use the option --locale. There are also individual options --lc-* and --icu-locale (see below) to set values for the individual locale categories. Note that inconsistent settings for different locale categories can give nonsensical results, so this should be used with care.

Alternatively, initdb can use the ICU library to provide locale services by specifying --locale-provider=icu. 
To alter the default encoding, use the --encoding. 

Options:
- `--locale=locale`:
  - sets the **default locale** for the **database cluster**;
  - if this option is **not** specified, the **locale** is **inherited** from the **environment** that `initdb` runs in;
  - if `--locale-provider` is `builtin`, `--locale` or `--builtin-locale` **must** be specified and set to `C`, `C.UTF-8` or `PG_UNICODE_FAST`;
- `--encoding=encoding` or `-E encoding`:
  - sets the **encoding** of the **template databases**;
  - **by default**, the *template database* **encoding** is **derived from** the **locale**;
  - if `--no-locale` is specified, then the **default** is `UTF8` for the `ICU` **provider** and `SQL_ASCII` for the `libc` **provider**.
  - this **encoding** will also be the **default encoding** of **any** database you create later, **unless** you **override** it in `CREATE DATABASE` statement;
- `--lc-collate=locale`:
  - sets the **locale** for the **specified category**;
- `--lc-ctype=locale`:
  - sets the **locale** for the **specified category**;
- `--lc-messages=locale`:
  - sets the **locale** for the **specified category**;
- `--lc-monetary=locale`:
  - sets the **locale** for the **specified category**;
- `--lc-numeric=locale`:
  - sets the **locale** for the **specified category**;
- `--lc-time=locale `:
  - sets the **locale** for the **specified category**;
`--no-locale`:
  - equivalent to `--locale=C`;
- `--builtin-locale=locale `
  - sets the **locale name** when the `builtin` **provider** is **used**;
- `--locale-provider={builtin|libc|icu}`
  - this option sets the **locale provider** for databases created in the **new** cluster;
  - the **default** is `libc`;
  - it **can be overridden** in the `CREATE DATABASE` command;

<br>

**Example**:
- `initdb --locale=fr_CA --lc-monetary=en_US`

<br>

## Configure encoding and locale using `CREATE DATABASE`
```sql
CREATE DATABASE db_name
WITH
   [TEMPLATE [=] template]
   [ENCODING [=] encoding]
   [LOCALE [=] locale ]
   [LOCALE_PROVIDER [=] locale_provider ]
   [BUILTIN_LOCALE [=] builtin_locale ]
   [ICU_LOCALE [=] icu_locale ]
   [LC_COLLATE [=] collate]
   [LC_CTYPE [=] ctype]
```

<br>

**Parameters**:
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

<br>

**Examples**:
- **Create** a new **database** with **custom encoding**:<br>
```sql
CREATE DATABASE mydb1_new_encoding WITH ENCODING 'UNICODE' TEMPLATE=template0;
```
- **Create** a new **database** with **custom encoding, collate and ctype**:<br>
```sql
CREATE DATABASE korean WITH ENCODING 'EUC_KR' LC_COLLATE='ko_KR.euckr' LC_CTYPE='ko_KR.euckr' TEMPLATE=template0;
```

<br>

# Collation in pg
In postgresql, *collation rules* can be applied at **3 levels**:
- **db** level (it **cannot** be altered after db was created);
- **column** level (**can** be altered);
- **query** level;

<br>

PostgreSQL provides **three** primary providers:
- **postgresql internal provider** (`c`): this is **built-in** collation support, it is **OS agnostic**;
- **system library provider** (`l`): uses GNU C library (**glibc**) and hence is **OS dependent**;
- **ICU provider**, **International Components for Unicode** (`i`): uses **ICU library**;

Run following command to check all collation strategies:
```sql
SELECT * FROM pg_collation;
```

<br>

The **provider** (e.g. **icu**) gives context about **how to interpret the locale value** given.<br>
For example both the `CREATE COLLATION`commands may use the same provider, but have two separate locale values: `en-u-kn-true` and `en@colNumeric=yes`.
```sql
CREATE COLLATION numeric (provider = icu, locale = 'en-u-kn-true');
CREATE COLLATION numeric (provider = icu, locale = 'en@colNumeric=yes');
```

<br>

Once the collation is created, it can be applied to the columns in table:
```sql
ALTER TABLE my_table ALTER COLUMN my_column TYPE character varying(255) COLLATE numeric;
```

<br>

Example of collations in PG:
- **C** collation
  - handled by **postgresql internal provider**
  - produces **predictable** and **deteministic** results **across** postgres instances;
- **en_US.UTF-8** collation
  - handled by **glibc**
  - produces **non-deteministic** results **across** postgres instances, because depends on OS locale settings;

<br>

**ICU collation names** are defined in the **Unicode Technical Standard #35**.<br>
**ICU collation name** consist of **subtags** separated by `-` or `_`.<br>

For example, the **collation** `u-kn-true` **enables** a **numeric ordering within alphanumeric strings**.

The **subtags** are:
- **language subtag**
- (optional) **script subtag** (describes alphabet that is used)
- (optional) **region subtag**
- (optional) `-u-` is a **tag extension**, :
  - `kn`: if `true` it means **sequences of digits** will be **ordered numerically** rather than alphabetically (**natural ordering**);
  - ...

<br>

## Examples
```sql
foo=# SELECT * from unnest(
   ARRAY(SELECT name COLLATE "C" FROM foo ORDER BY name),
   ARRAY(SELECT name COLLATE "default" FROM foo ORDER BY name),
   ARRAY(SELECT name COLLATE "POSIX" FROM foo ORDER BY name),
   ARRAY(SELECT name COLLATE "en_US.UTF-8" FROM foo ORDER BY name)
) AS data(C, "default", POSIX, "en_US.UTF-8")
;
     c     |  default  |   posix   | en_US.UTF-8
-----------+-----------+-----------+-------------
 1a1       | 1a1       | 1a1       | 1a1
 1a2       | 1a2       | 1a2       | 1a2
 1b        | 1b        | 1b        | 1b
 1b1       | 1b1       | 1b1       | 1b1
 1b2       | 1b2       | 1b2       | 1b2
 1ba       | 1ba       | 1ba       | 1ba
 2a        | 2a        | 2a        | 2a
 2a1       | 2a1       | 2a1       | 2a1
 2a2       | 2a2       | 2a2       | 2a2
 2b1       | 2b1       | 2b1       | 2b1
 2b2       | 2b2       | 2b2       | 2b2
 A_1.1.1.1 | a_1.1.1.1 | A_1.1.1.1 | a_1.1.1.1
 A_2.2.2.2 | A_1.1.1.1 | A_2.2.2.2 | A_1.1.1.1
 B_1.1.1.1 | a_2.2.2.2 | B_1.1.1.1 | a_2.2.2.2
 B_2.2.2.2 | A_2.2.2.2 | B_2.2.2.2 | A_2.2.2.2
 a1        | a1        | a1        | a1
 a11       | a11       | a11       | a11
 a12       | a12       | a12       | a12
 a2        | a2        | a2        | a2
 a22       | a22       | a22       | a22
 a_1.1.1.1 | aaabc     | a_1.1.1.1 | aaabc
 a_2.2.2.2 | aabb      | a_2.2.2.2 | aabb
 aaabc     | aabbcc    | aaabc     | aabbcc
 aabb      | ab        | aabb      | ab
 aabbcc    | abc       | aabbcc    | abc
 ab        | b_1.1.1.1 | ab        | b_1.1.1.1
 abc       | B_1.1.1.1 | abc       | B_1.1.1.1
 b_1.1.1.1 | b_2.2.2.2 | b_1.1.1.1 | b_2.2.2.2
 b_2.2.2.2 | B_2.2.2.2 | b_2.2.2.2 | B_2.2.2.2
 ba        | ba        | ba        | ba
(30 rows)

Time: 6.898 ms
foo=#
```