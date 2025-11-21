# Join types and methods
There are several **types of joins**:
- **inner** joins: `INNER JOIN`, or simply `JOIN`;
- **outer** joins: `LEFT OUTER JOIN`, `RIGHT OUTER JOIN`, `FULL OUTER FOIN`, or simply `LEFT JOIN`, `RIGHT JOIN`, `FULL FOIN`;

All these joins above are **logical operations** in the SQL queries.<br>

**Join methods** are algorithms that implement various **types of joins** under the hood.<br>

<br>

Postgres provides several **join methods**:
- **nested loop**;
- **hash join**;
- **merge join**;

<br>

All join methods provide **parallel** mode:
- **parallel nested loop**;
- **parallel hash join**;
- **parallel merge join**;

<br>

# Nested loop
The basic algorithm of the **nested loop join** as follows:
- the **outer loop** traverses all the rows of the **outer set**;
- for each row in **outer set**, the nested loop goes through the rows of the **inner set** to find the ones that satisfy
the join condition;
  - the found matches are returned to the **parent node**;

<br>

The algorithm accesses the **inner set** as many times as there are rows in the **outer set**.<br>
A **nested loop** join is the most efficient way to find a **Cartesian product**.<br>

The `Nested Loop` **node** of query plan always has **2 child nodes**:
- the **upper** one represents the **outer set** of rows;
- the **lower** one represents the **inner set**;

<br>

Example of `Nested Loop` **node**:
![nested_loop](/img/nested_loop.png)

In this example, the **inner set** is represented by the `Materialize` **node**.<br>
The rows are accumulated in memory until their total size reaches **work_mem**, then PostgreSQL writes them into a temporary file on disk.<br>

<br>

### Cost estimation
The **startup cost** of the **nested loop join** operation combines the startup costs of **all** child nodes:<br>

![nested_loop_cost](/img/nested_loop_cost.png)

<br>

# Hash join
The basic algorithm of the **hash join** as follows:
- the `Hash Join` *node* calls the `Hash` *node*, which places the *whole* **inner set** of rows into a **hash table**;
  - PostgreSQL chooses an **inner set** of rows the set with minum rows.<br>
- for each row in **outer set**, the `Hash Join` *node* look up value in the **hash table**;
  - the found matches are returned to the **parent node**;

<br>

**Example** of `Hash Join` *node*:
![hash_join](/img/hash_join.png)

The **startup cost** of the join is not zero and equal to the **cost of hash table creation**.<br>

<br>

### Cost estimation of `Hash Join` *node*:
![hash_join_cost](/img/hash_join_cost.png)

**Note**, that `Seq Scan` *node* has **zero cost** because it doesn't need to do any **processing** before it starts output rows, in other words it doesn't have any overhead and can retrun rows immediatly.<br>

<br>

# Merge join
It is like merge sort: merge sorted sets, the **inner set** of rows and **outer set** must be sorted by join columns.<br>
Since the join algorithm **stops** as soon as **one of** the sets is **over**.

**Example** of `Merge Join` *node*:
![merge_join](/img/merge_join.png)

The startup cost of the join includes at least the startup costs of all the child nodes.