# Clustered index
A **heap** is a table where data rows are stored on the disk in an **unordered manner**.<br>
**Physical** storage order of rows in **heap** corresponds to **order** in which they **were inserted**.<br>
Such tables are called **unclustered tables**.<br>

<br>

**Clustering** is the process of **physically reordering** a table's rows **based** on the **index** and such index becomes **clustered index**.<br>

The is SQL command `CLUSTER` to **cluster** a table **according to an index**:
```sql
CLUSTER employees USING employees_ind;
```

<br>

**After** clustering **order** of tuples in the table **becomes the same** as the **order** of tuples in **clustered index**.<br>
When a table has a **clustered index**, the table is called a **clustered table**.<br>

A **clustered table** is a table that **physically** stores the data rows on the disk in the **same order** as the key values in its **clustered index**.<br>
There can be **only one** *clustered index* **per** *table*, because the **rows** themselves **can only be physically stored in one order**.<br>

<br>

**Drawbacks** of clustered index:
- **writing** to a table with a **clustered table** can be **slower**, because it requires to rearrange the data in the table;

<br>

So, there are 2 types of indexes:
- The **clustered index** is always **primary index**.
- The **non clustered index** is always **secondary index**.

The **secondary index** have a structure that is **separated** from the data rows and contains **sorted tuples**: `(value [of indexed column], pointer [to the row])`.
