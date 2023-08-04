# Data type
**Data type** (or simply **type**) is usually specified by
- a set of **allowed** values;
- a set of **allowed operations** on these values;
- a set of **constraints** on these values;
- a **representation** of these values as machine types (**memory layout**).

<br>

# Tuple
A **tuple** is a **finite ordered sequence** (list) of **elements**.
The number of **elements** in **tuple** is called the **degree**, e.g., `n-tuple` refers to a **tuple** of degree `n` (`n ≥ 0`).<br>

<br>

# Relation
Consider N **sets**:
```
D1 = {d11, d12, ... , d1K}
D2 = {d21, d22, ... , d2L}

...

DN = {dN1, dN2, ... , dNM}
```

<br>

A **relation** `R` over set of sets `D = {D1, D2, ..., Dn}` is a **set of tuples** `(d1, d2, ..., dn)`, where each **data element** (or simply **element**) `dj` is a member of set `Dj`.<br>
Sets `D1, D2, ..., Dn` are called **domains** or **attribute names** (or simply **attributes**) of relation `R`.<br>
In other words, **attributes** are **named arguments** of relation.<br>
Actually, every set `Dj` contains **allowed values** for appropriate element `dj` in tuple. So, **domain** (**attribute**) is effectively a **data type**.<br>
Each **element** `dj` in tuple is a **value** of some **attribute** (**domain**) `Dj`. So, **element** is also termed an **attribute value**.<br>
A **heading** of relation `R` is a **tuple** of **all** its **attributes names**, e.g., `(D1, D2, ..., Dn)`.<br>


<br>

# DBMS
A **relation** is a **two**-dimensional **table**.<br>
A **column** of table is an **attribute** (**domain**).<br>
A **row** of table is a **tuple** or **instance of entity**. So, **row** is a **type of entity**.<br>
A **relation schema** is a **heading** and **set of constraints** defined in terms of this *heading*.<br>
