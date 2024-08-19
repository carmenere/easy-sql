# Table of contents
- [Table of contents](#table-of-contents)
- [What is relation?](#what-is-relation)
  - [Data type](#data-type)
  - [Tuple](#tuple)
  - [Relation](#relation)
  - [In RDBMS](#in-rdbms)
- [Functional dependency](#functional-dependency)
  - [Regular vs. irregular](#regular-vs-irregular)
  - [Valid and not valid FD](#valid-and-not-valid-fd)
    - [Case 1](#case-1)
    - [Case 2](#case-2)
    - [Case 3](#case-3)
    - [Case 4 (not valid FD)](#case-4-not-valid-fd)
- [Types of FD](#types-of-fd)
  - [Trivial FD](#trivial-fd)
    - [Examples](#examples)
  - [Non-trivial](#non-trivial)
    - [Example](#example)
  - [Partial dependency](#partial-dependency)
  - [Multivalued FD](#multivalued-fd)
    - [Example](#example-1)
  - [Transitive FD](#transitive-fd)
    - [Example](#example-2)
  - [Fully functional dependency](#fully-functional-dependency)
    - [Example](#example-3)
- [Types of keys](#types-of-keys)

<br>

# What is relation?
## Data type
**Data type** (or simply **type**) is usually specified by
- a set of **allowed** values;
- a set of **allowed operations** on these values;
- a set of **constraints** on these values;
- a **representation** of these values as machine types (**memory layout**).

<br>

## Tuple
A **tuple** is a **finite ordered sequence** (list) of **elements**.
The number of **elements** in **tuple** is called the **degree**, e.g., `n-tuple` refers to a **tuple** of degree `n` (`n ≥ 0`).<br>

<br>

## Relation
Consider N **sets**:
```
D1 = {d11, d12, ... , d1K}
D2 = {d21, d22, ... , d2L}

...

DN = {dN1, dN2, ... , dNM}
```

<br>

A **relation** `R` over set of sets `D = {D1, D2, ..., Dn}` is a **set of tuples** `(d1, d2, ..., dn)`, where each **data element** (or simply **element**) `dj` is a member of set `Dj`.<br>
Sets `D1, D2, ..., Dn` are called **domains** or **attribute names** (or simply **attributes**) of *relation* `R` and *relation* `R` is denoted as `R(D1, D2, ..., Dn)`.<br>
In other words, **attributes** are **named arguments** of relation.<br>
Actually, every set `Dj` contains **allowed values** for appropriate element `dj` in tuple. So, **domain** (**attribute**) is effectively a **data type**.<br>
Each **element** `dj` in tuple is a **value** of some **attribute** (**domain**) `Dj`. So, **element** is also termed an **attribute value**.<br>
A **heading** of relation `R` is a **tuple** of **all** its **attributes names**, e.g., `(D1, D2, ..., Dn)`.<br>


<br>

## In RDBMS
A **relation** is a **two**-dimensional **table**.<br>
A **column** of table is an **attribute** (**domain**).<br>
A **row** of table is a **tuple** or **instance of entity**. So, **row** is a **type of entity**.<br>
A **relation schema** is a **heading** and **set of constraints** defined in terms of this *heading*.<br>

<br>

# Functional dependency
In math:
- **11** --> **function** --> **aa**;
  - **11** is an **input**;
  - **aa** is an **output**;

<br>

Consider **relation** `R(A,B)`, where `A` and `B` are **atributes** of relation `R`:
|`A`|`B`|
|:--|:--|
|1|a|
|2|b|
|3|c|

<br>

We see that if we know `A` we know `B`. In other words:
- value of `A` **determines** value of `B`;
- value of `B` **is determined** by value of `A`;

<br>

So, `A -> B` is a **functional dependancy** (**FD**).<br>

<br>

The **functional dependency** is a **relationship** that exists **between attributes** under some relation `R`.<br>
The **functional dependency** is denoted as `{X} → {Y}` or `X → Y`, where `X` and `Y` are **attributes**. Functional dependency is **directional**: `X → Y` does **not** mean than `Y → X`.
- the **attribute** on the **left** side (`X`) is called **determinant**;
- the **attribute** on the **right** side (`Y`) is called the **dependent**.

<br>

Variants to read notation `X → Y`:
- `X` functionally determines `Y`;
- `Y` is functionally dependent on `X`;

<br>

## Regular vs. irregular
There can be **more than one** attribute on the **left** side, but there is always **only one** attribute on the **right** side. But we can **combine** several functional dependencies `{A, B} → {Q}` and `{A, B} → {W}` **into one**: `{A, B} → {Q, W}`. So, `{A, B} → {Q, W}` is really shorthand for **two** functional dependencies.<br>

A functional dependency `X → Y` is **regular** if `Y` contains only a **single** attribute.<br>
A functional dependency `X → Y` is **irregular** if `Y` contains **more than one** attributes.<br>

<br>

For example, `AB → C` is **regular**, but `AB → CD` is **irregular**, where `A`, `B`, `C`, and `D` are attributes.<br>

<br>

## Valid and not valid FD
### Case 1
|Determinant|Dependent|
|:--|:--|
|1|a|
|1|a|

<br>

When there are the **same** *determinants* and they all have the **same** corresponding *dependents* such a case is a **valid** FD.<br>

<br>

### Case 2
|Determinant|Dependent|
|:--|:--|
|1|a|
|2|b|

<br>

When there are the **different** *determinants* and they all have the **different** corresponding *dependents* such a case is a **valid** FD.<br>

<br>

### Case 3
|Determinant|Dependent|
|:--|:--|
|1|a|
|2|a|

<br>

When there are the **different** *determinants* and they all have the **same** corresponding *dependents* such a case is a **valid** FD.<br>

<br>

### Case 4 (not valid FD)
|Determinant (e.g. `Name`)|Dependent (e.g. `Age`)|
|:--|:--|
|Alice|45|
|Alice|17|

In the table above, *attribute* `Name` **cannot uniquely** identify *attribute* `Age`, because **names may be repeated**.

<br>

When there are the **same** *determinants* and they all have **different** corresponding *dependents* such a case is **not** *valid* FD.<br>

<br>

<br>

# Types of FD
Types of FD:
- **trivial** FD;
- **non-trivial** FD;
- **partial** FD;
- **multivalued** FD;
- **transitive** FD;
- **fully** FD;

<br>

## Trivial FD
Consider FD `X → Y`, if a *dependent* is a **subset** of the *determinant*, i.e. `Y ⊆ X`, then it is called a **trivial** FD.<br>
**Trivial** FD doesn't provide any new information about the relations between attributes.<br>

Note, **every attribute** `X` is **trivially** depends on **itself**: `{X} → {X}`.<br>

<br>

### Examples
- `A → B`
- `{Id, Name} → {Name}`
- `{A,B} → {A}`
- `{A,B,C} → {A,C}`

<br>

## Non-trivial 
Consider FD `X → Y`, if a *dependent* **isn't subset** of the *determinant* (`Y ⊈ X`), then it is called a **non-trivial** FD.<br>
FD `X → Y` is **completely non-trivial** *iif* `Y ∩ X = ∅`.

<br>

### Example
- `{id, name} → {age}` **completely non-trivial**, because `{id, name} ∩ {age} = ∅`;
- `{A} → {B}` **completely non-trivial**;
- `{A,C} → {B,C}` **non-trivial**, because `{B,C} ⊈ {A,C}`;

<br>

## Partial dependency
Consider **relation** `R(A,B,C,D)`:
|A|B|C|D|
|:-|:-|:-|:-|
|1|22|d|f|
|2|33|q|m|

And consider **2 FD** in this relation `R`:
- `{A,B} -> {D}`;
- `{B} -> {C}`;

<br>

Also we know that `{A} -> {A}` and `{B} -> {B}`, because **every attribute** is **trivially** depends on **itself**.<br>

So, we can rewrite all above as `{A,B} -> {A,B,C,D}`.<br>

We see that `A` and `B` **together** is a **candidate key** `{A,B}`, because they both determine **all other** attributes in relation `R`.<br>

We can **differentiate** all atributes in 2 types:
- **prime** attributes;
- **non-prime** attributes;

<br>

Attributes that are **part** of **candidate key** are called **prime**.<br>
Attributes that are **not part** of **candidate key** are called **non-prime**.<br>

<br>

In example above,
- attributes `A` and `B` are **prime**;
- attributes `C` and `D` are **non-prime**;

<br>

There are 2 FD in example above, the FD `{B} -> {C}` is called **partial** FD because `B` is **not** a *candidate key* but it is a **part** of *candidate key* `{A,B}`.<br>

**Partial** FD occurs when at least one **non-prime** attribute depends on the **part** (**proper subset**) of **candidate** key, **not** complete **candidate** key.<br>

How to find partial dependency:
1. Identify **all candidate keys**.
2. Defferntiate **prime** and **non-prime** attributes.
3. Find **non-prime** attribus that are **determined by** a **part** of any **candidate** key.

<br>

## Multivalued FD
**Formal definition**: let $R$ be a relation schema and let $α ⊆ R$ and $β ⊆ R$ be sets of attributes. The **multivalued dependency** holds on $R$ if, for all pairs of tuples $t_1$ and $t_2$ such that $t_1[α] = t_2[α]$, there exist tuples $t_3$ and $t_4$ such that:
- $t_1[α] = t_2[α] = t_3[α] = t_4[α]$
- $t_1[β] = t_3[β]$
- $t_2[β] = t_4[β]$
- $t_1[R-(α ∪ β)] = t_4[R-(α ∪ β)]$
- $t_2[R-(α ∪ β)] = t_3[R-(α ∪ β)]$

<br>

The **multivalued dependency** is denoted as $α ↠ β$ and read as $α$ **multidetermines** $β$.<br>

A **multivalued dependency** $A ↠ B$ is **trivial** if $A ∪ B$ is the whole set of attributes of the relation, in other words, $A ∪ B = R$.<br>

The **multivalued dependency** can be schematically depicted as shown below:
|tuple|$α$|$β$|$R-(α ∪ β)$|
|:----|:--|:--|:----------|
|$t_1$|$a_1..a_n$|$b_1..b_m$|$d_1..d_k$|
|$t_2$|$a_1..a_n$|$c_1..c_m$|$e_1..e_k$|
|$t_3$|$a_1..a_n$|$b_1..b_m$|$e_1..e_k$|
|$t_4$|$a_1..a_n$|$c_1..c_m$|$d_1..d_k$|

<br>

Consider some relation $R$. A **multivalued dependency** exists when there are **at least three attributes** (like $X$, $Y$ and $Z$) in a relation $R$ and for a value of $X$ there is a well defined set of values of $Y$ and a well defined set of values of $Z$. However, the set of values of $Y$ is independent of set $Z$ and vice versa.<br>

Let the tuple $(x,y,z)$ denotes all values of relation $R(X,Y,Z)$, then whenever the tuples $t_1=(a,b,c)$ and $t_2=(a,d,e)$ exist in relation $R$, the tuples $t_3=(a,b,e)$ and $t_4=(a,d,c)$ should also exist in the relation `R`.

<br>

### Example
- `{id} ↠ {name, age}`, there is **no** FD between `name` and `age`;

<br>

## Transitive FD
Consider 2 FDs: `A → B` and `B → C`, where `A` is a *candidate key*. There is also **3rd FD** `A → C` exists according to **axiom of transitivity**. Thus, *candidate key* `A` **indirectly** determines attribute `C` through `B`, that's why FD `A → C` is called **transitive** FD.<br>

<br>

### Example
Consider table `T`: 
|A|B|C|
|:-|:-|:-|

<br>

In this table following FD exist:
- `A → B`;
- `B → C`;

<br>

So, this table introduces **transitive dependency** `A → C`.

<br>

## Fully functional dependency
Consider FD `{X1, X2, ..., Xn} → Y`, if *dependent* (`Y`) **cannot** be **determined** by any **proper subset** of its *determinant* (`{X1}, {X1,X2}, {X2,X3}, {X2,X3,X4}, ... `) and it is **determined** only by full set of its *determinant* `{X1, X2, ..., Xn}`, then such FD is called a **fully functional** FD.<br>

<br>

### Example
- `{student_id, course_id} → {mark}`, since there is **no** FD between `student_id` and `mark` and between `course_id` and `mark`

<br>

# Types of keys
A **superkey** is a **set of attributes** that **uniquely** identifies **each tuple** of a relation.<br>
Because **superkey** values are **unique**, tuples with the **same** superkey value must also have the **same** **non-key** attribute values.<br>
That is, **non-key attributes** are functionally dependent on the **superkey**.<br>

A **trivial superkey** is the set of **all** attributes, because tuples in a relation are *by definition* **unique**.<br>

A **natural key** is a **superkey** that consists of values from **real** domains.<br>

A **surrogate key** (**synthetic key**, **entity identifier** is a unique identifier generated by the db or another software automatically.<br>

A **candidate key** (or **minimal superkey**) is a **superkey** that **can't** be reduced by removing an attribute.<br>
There can be **more** than **1** **candidate key** in relation.<br>

A **composite key** is a **candidate key** that consists of **two or more** attributes (table columns) that together **uniquely** identify an entity occurrence (table row).<br>

A **primary key** is the **candidate key** chosen to be **primary**. Usually it is **minimal** *candidate key*. Any other *candidate key* is an **alternate key**.<br>

A **prime attribute** of a relation is an attribute that is a member of **any** *candidate key*.<br>

A **non-prime attribute** of a relation is an attribute that **isn't** a **prime attribute**, i.e., a **non-prime attribute** of a relation is an attribute that is **not** a member of any **candidate key** of the relation.<br>
