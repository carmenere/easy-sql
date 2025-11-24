# What is relation?
A **relation** `R` is a **set of tuples** `(d1, d2, ..., dn)`, where each **element** `dj` is a member of set `Dj`.<br>
Sets `D1, D2, ..., Dn` are called **domains** or **attribute names** (or simply **attributes**) of *relation* `R` and *relation* `R` is denoted as `R(D1, D2, ..., Dn)`.<br>
In other words, **attributes** are **named arguments** of *relation*.<br>
Actually, every set `Dj` contains **allowed values** for appropriate element `dj` in tuple. So, **domain** (**attribute**) is effectively a **data type**.<br>
Each **element** `dj` in tuple is a **value** of some **attribute** (**domain**) `Dj`. So, **element** is also termed an **attribute value**.<br>
A **heading** of relation `R` is a **tuple** of **all** its **attributes names**, e.g., `(D1, D2, ..., Dn)`.<br>

A **degree** of *relation* (**arity**) is a **number of attributes** in a *relation*.<br>
A **cardinality** of *relation* is a **number of tuples** in a *relation*.<br>

There is a **hierarchy**: `relation` -> `tuple` -> `attribute`.<br>

The definitions **relation**, **tuple**, **attribute** are formal definitions from relational theory.<br>

Mapping to other unformal defenitions:
- `file` -> `record` -> `field`;
- `table` -> `row` -> `column`;

<br>

So, in RDBMS:
- a **relation** is a **two**-dimensional **table**;
- a **row** of table is a **tuple** or **instance of entity**;
- a **column** of table is an **attribute** (**domain**);
- a **schema** of *relation* is a **heading** of *relation* and **set of constraints**;
