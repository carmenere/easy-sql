# Entity Relationship Model
A basic  is composed of entities and relationships between them.<br>
The **Entity Relationship Model** (**ER model**) is an **abstract data model**, that illustrates the **relationships** between **entities**.<br>
**ER model** defines an **abstract data model** which can be implemented in a database, typically a *relational database*.<br>
**Entity Relationship Diagram** (**ERD**) is a graphical representation of **ER model**.<br>

There are various ERD notation exist:
- **Crow's foot** notation (the most popular one);
- **UML** notation;
- **IDEF1X** notation;
- **Bachman** notation;
- **Min-Max / ISO** notation;
- **Chen** notation;

<br>

# Crow's foot notation
## Entities
An **entity** is represented by a **rectangle**, with its name on the top and list of its **attributes**.<br>
An **attribute** is a property that describes a particular entity.<br>
An **identifier** is an attribute uniquely distinguishes an instance of the entity. 

<br>

Notations:
- `#` is for **identifier**;
- `*` is for **mandatory** attribute;
- `○` is for **optional** attribute;

<br>

```sh
+------------+
| SomeEntity |
+------------+
|  #*Attr1   |
|   *Attr2   |
|   *Attr3   |
|   ○Attr4   |
|    ...     |
+------------+
```

<br>

## Relationships
**Relationships** illustrate the association between two entities and is represented by straight line.<br>
Every *relationship* is **bidirectional**.<br>

Every relationship has
- **names** at both ends;
- **cardinality** at both ends (for every direction);
- **modality** at both ends (for every direction);
- **transferability**;

<br>

```plain
cardinality  modality (it is right to left modality/cardinlity, for Address)
         \    /
 +------+ \  /            +---------+
 | Name |--⚞○---------○|--| Address |
 +------+            /  \ +---------+  
                    /    \
              modality  cardinality (it is left to right modality/cardinlity, for Name)
```

<br>

So, there are **two** modality/cardinlity in relationship:
- modality/cardinlity of **left to right** direction, it is placed *next* to the **right** entity (`Address`);
- modality/cardinlity of **right to left** direction, it is placed *next* to the **left** entity (`Names`);

<br>

Every relationship in ER diagrams can be read both **from left to right** and then **from right to left**.<br>
For above example: 
- **from left to right**: every **one** row in table `Names` can be associated with **zero** or **exactly one** row in table `Address`;
- **from right to left**: every **one** row in table `Address` can be associated with **zero**, **one** or **many** rows in table `Names`;

<br>

While reading relationship from **from left to right** zero modality means 

## Cardinality. Modality
**Cardinality** and **Modality** are the indicators of the business rules around a relationship.<br>

### Modality
**Modality** refers to the **minimum** number of times an instance (row) in one entity (table, relationship) can be associated (connected) with an instance (row) in the related entity.<br>
**Modality** expresses **optionality** of relationship for particular direction.<br>
- `○` means **minimum 0** (means **optional**);
- `|` means **minimum 1** (means **mandatory**)

<br>

### Cardinality
**Cardinality** refers to the **maximum** number of times an instance (row) in one entity (table, relationship) can be associated (connected) with instances (rows) in the related entity.<br>
**Cardinality** can be:
- `|` means **maximum 1**;
- `⚟` or `⚞` means **maximum is unlimited** or **many** or **more than zero** (**crow’s foot** sign)

<br>

There are 4 combinations for modality/cardinlity for one direction:
|Combination|Crow's foot notation|UML notation|
|:----------|:-------------------|:-----------|
|**0** or **1**|`--○\|--`|`0..1`|
|**Exactly 1**|`--\|\|--`|`1`|
|**0** or **More**|`--○⚟--`|`0..*`|
|**1** or **More**|`--\|⚟--`|`1..*`<br>UML also supports specific range `N..M`|

<br>

## Parent. Child
Cardinalities on both ends of line defines 3 **relationship types**:
|Left cardinality (near Name)|Right cardinality (near Address)|Relationship type|
|:---------------|:-------------------|:------------|
|`--\|--`|`--\|--`|**1:1** (One-to-One)|
|`--\|--`|`--\⚟--`|**1:M** (One-to-Many)|
|`--⚞--`|`--\|--`|**M:1** (Many-to-One)|
|`--⚞--`|`--⚟--`|**M:M** (Many-to-Many)|

<br>

Relations **1:M** and **M:1** are dependent direction, but semantically the same and called **1:M**.<br>

<br>

### 1:M
For **1:M** relationship:
- entity that is next to `1` (`|`) is called **parent**;
- entity that is next to `M` (`⚟`) is called **child**;

<br>

Consider example of **1:M** relationship:
```
+------+              +---------+
| Name |--||------|⚟--| Address |
+------+              +---------+
```

Here:
- `Name` is **parent**;
- `Address` is **child**.

<br>

### M:M
**M:M** relationships are usually implemented by means of an **associative table** (aka **join table**, **junction table**).<br>
In other words, **M:M** is two **1:M** relationships.<br>

Consider example of **M:M** relationship:
```
+--------+               +------+
| Author |--⚞|-------|⚟--| Book |
+--------+       |       +------+
                 |
        +-----------------+
        |       a2b       |
        +-----------------+
        | FK to author_id |
        |  FK to book_id  |
        +-----------------+
```

To implemet **M:M** relationship between `Author` and `Book` **3rd** table `a2b` is needed and **M:M** relationship relationship splits into:
- **1:M** realtion relationship `Author` -> `a2b`;
- **1:M** realtion relationship `Book` -> `a2b`;

<br>

In this case the **primary key** for `a2b` is formed from the 2 **foreign keys**: to `author_id` and to `book_id`.<br>

<br>

## Modality
Consider relationship:
```
+--------+              +-------+
| parent |--|○------○⚟--| child |
+--------+              +-------+
```

<br>

Modality near **child** realated to `parent -> child` direction of relationship:
- `○` means that query `... FROM parent LEFT JOIN child ON parent.id = child.fk_parent_id ... ` **can** contains `NULL` in right part of join, i.e., there can be such `parent.id` that are **not** connected to any *child* (`NULL`);
- `|` means that query `... FROM parent LEFT JOIN child ON parent.id = child.fk_parent_id ... ` **can’t** contains `NULL` in right part of join, i.e., there **can’t** be such `parent.id` that are **not** connected to any *child* (`NULL`);

<br>

Modality near **parent** is realated to `child -> parent` direction of relationship:
- `○` means **FK** from *child* to *parent* **can** be `NULL`;
- `|` means **FK** from *child* to *parent* **can't** be `NULL`;

<br>

## Transferability
Transferability answers to question: can child change its parent?
- if child **can** change its *parent* (rewrite **FK** to another `parent id`) - then relationship is **transferable**;
- if child **can’t** change its *parent* (rewrite **FK** to another `parent id`) - then relationship is **non-transferable**.
