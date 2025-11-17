# Locale
A **locale** is a **set of language and cultural rules**.<br>
A name of locale has a format: **language code**_**COUNTRY CODE**.<br>
For example, the locale `zh_CN` has two parts: `zh` is the **language code**, and `CN` is the **country code**.<br>

The `locale` command, when called **without argument** gives a summary of the current settings.<br>
There are **different categories** for locale information a program might need, **locale settings**.<br>
There are several **locale variables** for **locale settings**:
- `LC_COLLATE` controls the **collation**;
- `LC_CTYPE` determines
  - the **interpretation of byte sequences as characters** (e.g., single versus multibyte characters);
  - the **character classifications** (e.g., alphabetic or digit);
  - the **behavior of character classes**;
- `LC_MONETARY` Format for monetary amounts;
- `LC_NUMERIC` Format for numbers;
- `LC_TIME` Format for dates and times;
- `LC_ALL` if set **overrides** all the other localisation settings;

<br>

The **C locale** is a **special locale** that is meant to be the **simplest locale**.<br>
In the **C locale**:
- **characters are single bytes**;
- the **charset is ASCII**;
- the **sorting order** is based on the **code point values** (**byte values**);
- the **language** is usually **US English**;
- and things like _currency symbols_ are **not defined**;

<br>

# Collation
**Collation** is a **set of rules** used for **sorting** and **comparison properties**.<br>

Usually **code point values** are **used** as a **default collation** - `A` with code point `65` is **before** `a` with code point `97`.<br>
**Sort algorithm** using **different** collations can produce **different** result for the **same** input strings.<br>

<br>

**Example 1**: What is correct?

1. 
```bash
a.b
a.c
ad
ae
a.f
a.g
```

**OR**

2. 
```bash
a.b
a.c
a.f
a.g
ad
ae
```

<br>
<br>

**Example 2**: What is correct?

1. 
```bash
#
$
@
^
&
```

**OR**

2. 
```bash
@
&
#
^
$
```

<br>

# Collation in pg
In postgresql, *collation rules* can be applied at **3 levels**:
- **db** level (it **cannot** be altered after db was created);
- **column** level (**can** be altered);
- **query** level;

<br>

PostgreSQL support collation support through three primary providers:
- **postgresql internal provider** (**c**): this is **built-in** collation support, it is **OS agnostic**;
- **system library provider** (**l**): uses GNU C library (**glibc**) and hence is **OS dependent**;
- **ICU provider**, **International Components for Unicode** (**i**): uses **ICU library**;

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

