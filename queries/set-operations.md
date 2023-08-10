# Combining queries
The results of two queries can be **combined** using the **set operations**.<br>

The syntax is
- **union**: `Q1 UNION [ALL] Q2`;
- **intersection**: `Q1 INTERSECT [ALL] Q2`;
- **difference** (`Q1 - Q2`): `Q1 EXCEPT [ALL] Q2`;

<br>

> **Notes**:<br>
> **Without** `ALL` keyword duplicate rows are **eliminated** from result.<br>
> **With** `ALL` keyword duplicate rows are **kept** in result.

<br>

