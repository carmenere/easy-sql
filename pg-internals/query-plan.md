# Query plan
PostgreSQL builds **query plan** before it executes query.<br>
*Query plan* is just **tree** that consist of **plan nodes**.<br>

<br>

To view query plan there is command:
- `EXPLAIN`: print out only **estimated data**;
- `EXPLAIN ANALYZE`: execute query and print out **also actual values**:
  - instead cost it shows **time in milliseconds**;
  - also it has value **loops** it show how many times this plane node was executed;
  - under the query plan it also outputs **planning time** and **execution time**;
- `EXPLAIN (COSTS OFF)`: print out **only tree** of query plan, without any estimations;

<br>

The **cost** is an **abstract measure** of **how long** a query will take to execute.<br>

<br>

Every node in query plan has **3 estimated values** based on the available statistics:
- **cost**: `cost=startup_cost..total_cost`
  - **startup_cost**: it represents the *estimated* amount of time (in *abstract units*) **between** when the *node* **starts** executing and **when** the *node* outputs its **first row**;
  - **total_cost**: it represents the *estimated* amount of time (in *abstract units*) **between** when the *node* **starts** executing and **when** the *node* outputs its **last row**;
- **rows**: `rows=N`: *estimated* number of all rows to be selected in the query;
- **width**: `width=K`: *estimated* average size of row in the result;

<br>

**Example**:
- that `Seq Scan` *node* has **zero cost** because it **doesn't** need to do any **processing** before it can output the **first** row, in other words it **doesn't** have any overhead and can retrun rows immediatly;
- that `Sort` *node* has **non-zero cost** because it needs to perform sorting before it can output the **first** row;

<br>

The **cost** estimation of the **upper node** is based on the **cost** of **all** its **child nodes**.<br>
For example, the **startup cost** estimation of the **upper node** includes **total cost** of its **child node** and the **cost** that **upper node** itself adds:<br>
![explain_cost_upper](/img/explain_cost_upper.png)

<br>

# Planner settings
- `SET enable_hashjoin TO off`, disables to use the **hash join**, the default is `on`;
- `SET enable_mergejoin TO off`, disables to use the **merge join**, the default is `on`;
- `SET enable_nestloop TO off`, disables to use the **nested loop**, the default is `on`;

<br>

In fact, `enable_XXXjoin TO off` sets **minimum** priority for appropriate join method, but planner is **able** to use it.<br>
