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
- every attribute in relation is **singled valued** attribute, i.e., relation **doesn't** contain any **composite** or **multi-valued** attribute;
- all tuples in relation are uniqie;

<br>

## 2NF
2NF removes **partial** dependency.<br>

A relation is in **2NF** iif:
- relation that is in **1NF**;
- there is **no** *partial dependency* in relation, i.e., all **non-key** attributes must depend on the **entire key**;

<br>

## 3NF
**3NF** removes **transitive** dependency.<br>

A relation is in **3NF** iif:
- relation that is in **2NF**;
- there is **no** *transitive dependency* in relation;

<br>

## BCNF
**BCNF** removes dependencies of **prime** attributes form **non-prime** attributes.<br>

A relation is in **BCNF** iif:
- relation that is in **3NF**;
- for any dependency `A → B`, `A` should be **super** or **candidate** key;

<br>

# 4NF
**4NF** removes **multi-valued** dependency.<br>

A relation is in **4NF** iif:
- relation that is in **BCNF**;
- there is **no** *multi-valued dependency* in relation;
