# Collation
The `locale` command, when called **without** **argument** gives a summary of the current settings.<br>
The `LC_COLLATE` controls the **collation**.<br>
The `LC_ALL` is the **environment variable** that overrides all the other localisation settings.<br>

**Collation** is an instruction on **how to compare two strings**.<br>
Usually **code point values** are **used** as a **default collation** - `A` with code point `65` is **before** `a` with code point `97`.<br>
**Sort algorithm** using **different** collations can produce **different** result for the **same** input strings.<br>

<br>

The **C locale** is a **special locale** that is meant to be the **simplest locale**.<br>
In the **C locale**:
- **characters are single bytes**;
- the **charset is ASCII**;
- the **sorting order** is based on the **code point values** (**byte values**);
- the **language** is usually **US English**;
- and things like _currency symbols_ are **not defined**;

<br>

## Example: Sorting order in different locales
The **US locale ignores** periods:
```bash
a.b
a.c
ad
ae
a.f
a.g
```

The **C locale doesn't ignore** periods:
```bash
a.b
a.c
a.f
a.g
ad
ae
```
