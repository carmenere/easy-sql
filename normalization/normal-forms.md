# Modification anomalies
**Redundancy** occurs when the **same** data value is stored **more than once** in a relation.<br>
**Redundancy** in relation may cause **insertion**, **deletion** and **update** anomalies:
- **insertion anomalies** - insertion of a row into the relation (table) cause to redundant information;
- **deletion anomalies** - deletion of a row may lose information that is still required to be stored;
- **modification anomalies** - changing an attribute of a row may require changing multiple attribute values in other rows.

<br>

Normalization **decomposes** removes unwanted redundancies and relation into smaller relations that contains less redundancy.<br>
**Normalization** is a process of sequentially transferring a relation from **UNF** (**un-normilized form**, **de-normalized** state) to a **higher order NF**.<br>

<br>

Steps to transfer relation to next NF:
1. Identify FDs.
2. Determine whether FDs meet NF conditions or not.
3. Split relation (table) to meet the NF conditions.

<br>

# Normal forms
There are 6 normal forms defined:
- **1NF** - First normal form;
- **2NF** - Second normal form;
- **3NF** - Third normal form;
- **BCNF** - Boyce-Codd normal form;
- **4NF** - Fourth normal form;
- **5NF** - Fifth normal form;
- **6NF** – Sixth normal form (its purpose is **not** to remove redundancy );

<br>

Each **NF** remove particular type of FD.<br>

<br>

## 1NF
A relation is in **1NF** iif:
- every attribute in relation is **singled valued** attribute, i.e., relation **doesn't** contain any **composite** or **multivalued** attribute;
- all tuples in relation are uniqie;

<br>

More formal: a relation is in **1NF** iif **no** attribute domain has relations as elements.

<br>

## 2NF
2NF removes **partial** dependencies.<br>

A relation is in **2NF** iif:
- relation is in **1NF**;
- there is **no** *partial dependency* in relation, i.e., every **non-prime** attribute of the relation is dependent on the **whole** of *every* **candidate** key;

<br>

Note that **2NF** **doesn't** put any restriction on the **non-prime** to **non-prime** attribute dependency.<br>

A **FD** on a **proper subset** of **any** **candidate** key is a **violation** of **2NF**, i.e., such relation is not in **2NF**.

<br>

### Example
Consider followong table:
|Manufacturer|Model|Manufacturer country|
|:-----------|:----|:-------------------|
|Forte|X-Prime|Italy|
|Forte|Ultraclean|Italy|
|Dent-o-Fresh|EZbrush|USA|
|Brushmaster|SuperBrush|USA|
|Kobayashi|ST-60|Japan|
|Hoch|Toothmaster|Germany|
|Hoch|X-Prime|Germany|

<br>

There is **one** **candidate key** is `{Manufacturer, Model}`.<br>
There is **FD** `{Manufacturer} -> {Manufacturer country}`.<br>

So,
- `{Manufacturer}` is **proper subset** of `{Manufacturer, Model}` **candidate key**;
- `{Manufacturer country}` is **not** part of a **candidate key**, because `{Manufacturer country, Model}` is not unique tuple, so it is a **non-prime attribute**;

<br>

That is, `{Manufacturer country}` (**non-prime attribute**) is functionally dependent on a **proper subset** `{Manufacturer}` of a `{Manufacturer, Model}` candidate key.<br>
The relation is in **violation** of **2NF**.<br>

To make the relation **conform** to **2NF**, it is necessary to split it into **two** relations:
- `{Manufacturer, Model}`
- `{Manufacturer, Manufacturer country}`

<br>

## 3NF
**3NF** removes **transitive** dependencies.<br>

**Codd's definition** states that a relation is in **3NF** iif **both** of the following conditions hold:
- relation is in **2NF**;
- all **non-prime** attributes depend **only** on the **primary key**, i.e., **no** **non-prime attribute** of relation is **transitively** dependent on the **primary key**.

<br>

**Carlo Zaniolo's definition** states that a relation is in **3NF** iif for **each** of its functional dependencies `X → Y`, **at least** **one** of the following conditions holds:
1. `Y ⊆ X`, i.e., `X → Y` is **trivial** FD;
2. `X` is a **superkey**;
3. **Every** element of `Y \ X` (**set difference**), is a **prime attribute**, i.e., **each** attribute in `Y \ X` is contained in some **candidate key**.

Zaniolo's definition gives a clear sense of the **difference** between **3NF** and **BCNF**:
- **3NF** allow the **right** side of the **FD** to be a **prime attribute**;
- **BCNF** simply eliminates the **third** alternative, i.e., **doesn't** allow the **right** side of the **FD** to be a **prime attribute**;

<br>

### Example

Tournament winners
|Tournament|Year|Winner|Winner's date of birth|
|:---------|:---|:-----|:---------------------|
|Indiana Invitational|1998|Al Fredrickson|21 July 1975|
|Cleveland Open|1999|Bob Albertson|28 September 1968|
|Des Moines Masters|1999|Al Fredrickson|21 July 1975|
|Indiana Invitational|1999|Chip Masterson|14 March 1977|

<br>

The **composite key** `{Tournament, Year}` is a **minimal** set of attributes guaranteed to uniquely identify a row. That is, `{Tournament, Year}` is a **candidate** key for the table.<br>
The **non-prime** attribute `Winner's date of birth` is **transitively** dependent on the **candidate** key `{Tournament, Year}` through the **non-prime** attribute `Winner`.<br>

To become **3NF** table must be splitted into 2 tables:
- `{Tournament, Year, Winner}`
- `{Winner, Date of birth}`

<br>

## BCNF
**BCNF** removes dependencies of **prime** attributes form **non-prime** attributes.<br>

A relation is in **BCNF** iif:
- relation is in **3NF**;
- **determinant** of **any** dependency in relation is a **candidate** key, i.e., for **any** dependency `A → B`, `A` must be **candidate** key;

<br>

Assume there is relation in **3NF**, it is  **NP-complete** to determine whether it violates **BCNF** or not.

<br>

# 4NF
**4NF** removes **multivalued** dependencies.<br>

A relation is in **4NF** iif:
- relation is in **BCNF**;
- there is **no** *multivalued dependency* in relation;

<br>

## Example
|Restaurant|Pizza Variety|Delivery Area|
|:---------|:------------|:------------|
|A1 Pizza|Thick Crust|Springfield|
|A1 Pizza|Thick Crust|Shelbyville|
|A1 Pizza|Thick Crust|Capital City|
|A1 Pizza|Stuffed Crust|Springfield|
|A1 Pizza|Stuffed Crust|Shelbyville|
|A1 Pizza|Stuffed Crust|Capital City|
|Elite Pizza|Thin Crust|Capital City|
|Elite Pizza|Stuffed Crust|Capital City|
|Vincenzo's Pizza|Thick Crust|Springfield|
|Vincenzo's Pizza|Thick Crust|Shelbyville|
|Vincenzo's Pizza|Thin Crust|Springfield|
|Vincenzo's Pizza|Thin Crust|Shelbyville|

<br>

Each row indicates that a given restaurant can deliver a given variety of pizza to a given area.<br>
The table has **no** non-key attributes because its only candidate key is `{Restaurant, Pizza Variety, Delivery Area}`. Therefore, it is in **BCNF**.<br>
But this relation has **two** non-trivial **multivalued** dependencies on the `{Restaurant}` attribute (which is **not** a superkey):
- `{Restaurant} → {Pizza Variety}`
- `{Restaurant} → {Delivery Area}`

<br>

This leads to redundancy in the table. To eliminate redundancy we must separate relation `{Restaurant, Pizza Variety, Delivery Area}` into 2 relations:
- `{Restaurant, Pizza Variety}`
- `{Restaurant, Delivery Area}`
