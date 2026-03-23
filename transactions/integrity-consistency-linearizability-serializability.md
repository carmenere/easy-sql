# Table of contents
<!-- TOC -->
- [Table of contents](#table-of-contents)
- [Integrity vs. Consistency](#integrity-vs-consistency)
- [Coherence vs. Consistency](#coherence-vs-consistency)
- [Consistency](#consistency)
  - [Consistency models](#consistency-models)
- [Linearizability vs Serializability](#linearizability-vs-serializability)
<!-- TOC -->

<br>

# Integrity vs. Consistency
**Integrity** ensures that the data remains **correct** and **valid** over its **entire lifecycle**, i.e. **from** *creation* **to** *deletion*.<br>
**Consistency** in distributed systems ensures that **all replicas** have the **same copy** of data, meaning **any read operation** returns the **most recent write**, **regardless** of which node performed it, **consistency** ensures that data are **synchronized across all replicas**. In **consistent systems**, every user or application sees the same, up-to-date information, **regardless** of which node/replica they query.<br>

So, data can be:
- **consistent but corrupted**, i.e. **all** replicas have **corrupted data**;
- **correct but inconsistent**, i.e. **one** replica has the **most recent** copy of data against other replicas;

<br>

The _key feature_ of relational databases is their ability to _ensure_ **data consistency** and **data integrity**:
- *integrity* is maintained through **specific business rules** and **constraints**
  - at the database level it is possible to create **integrity constraints**, e.g. `UNIQUE`, `NOT NULL`, **referential integrity constraints** and so on;
- *consistency* is maintained through **synchronization** *between replicas* and **ACID** *per one replica or individual db node's data*;

<br>

If all required constraints can be formulated at the database level, consistency would be guaranteed. But some conditions are too complex for that.<br>
If app breaks consistency without breaking the integrity, there is no way for db to detect such violations.<br>
Thus, data _consistency_ is **stricter** than _integrity_.<br>

<br>

**Transaction** is a **set of operation** that has the following **properties** (aka **ACID**):
- **C**onsistency: it transforms database from one **consistent** state to another **consistent** state;
- **A**tomicity: **all operations** are executed as a **single unit** of work or rolled backed;
- **I**solation: it **doesn't affect other transactions**;
- **D**urability: after crash, the system may still contain some changes made by uncommitted transactions and **the system must be able to restore data consistency after craches**;

<br>

So, **isolation levels** are **crucial** for maintaining *data consistency* and *integrity*.<br>

<br>

# Coherence vs. Consistency
**Coherence** is like consistency but it handles **single locations** whereas consistency handles multiple locations.<br>
**Coherence** ensures **all** nodes/processors across the system (distributed or not) see the **most resent version** (i.e. the **most recent write**) of **some data item** (e.g., **variable** `X`).<br>
Coherence typically refers to **cache coherence** and it is often **managed by hardware**. It uses protocols like **MESI** to manage cache line states.<br>

<br>

# Consistency
**Consistency model** defines **constraints** or **requirements** that **guarantees** that **data** will be **consistent** and the results of **reading**, **writing**, or **updating** will be **predictable**.<br>

One *consistency model* can be considered **stronger** than another if it requires **all constraints** of that *model* and more. In other words, a *model* with **fewer constraints** is considered a **weaker** *consistency model*.<br>

<br>

## Consistency models
[**Consistency models hierarchy**](https://jepsen.io/consistency/models):<br>
![consistency_models](/img/consistency_models.png)

<br>

At the top of hierarchy is the **strong serializability** (aka **strict serializability**) *consistency model*.<br>
**Strong serializability** combines **serializable** *isolation level* and **linearizable** *consistency level*.<br>

<br>

*Consistency models* **under** *strong serializability* are divided into **2 groups**:
- **isolation levels**;
- **consistency levels**;

<br>

*Consistency models* **under** *strong serializability*:
- **isolation levels**:
  - **read uncommitted**;
  - **read committed**;
  - **repeatable read**;
  - **serializable**;
- **consistency levels**:
  - **linearizable consistency**
  - **sequential consistency**
  - **causal consistency**
  - **eventual consistency**

<br>

# Linearizability vs Serializability
**Linearizability** is a **guarantee** about *single operations* on *single objects*.<br>
**Serializability** is a **guarantee** about *transactions* (*groups of one or more operations*) over *multiple objects* (*one or more objects*).<br>

*Linearizability* for read and write operations refers to the **C** or **consistency** in the **CAP theorem**.<br>

It **guarantees** that the execution of *multiple concurrent* (happening at the same time) *transactions* over *multiple objects* is **equivalent to some serial execution** (total ordering) of the transactions.<br>

In other words **serializability** ensures that these transactions play out as if they happened **one after another**, **not all at once**.
This doesn't mean they physically occur one by one - they can still happen all at once. But the **final result** *will be the same as if they happened sequentially*.<br>
*Serializability* refers to the **I** or **isolation**, in **ACID**.<br>

<br>

- **linearizability** is one of the **strongest single-object** *consistency model*:
  - it implies that every operation appears to take place **atomically**, **in some order**, consistent with the real-time ordering of those operations;
    - e.g., if operation `A` completes **before** operation `B` begins, then `B` must see the result of the `A`;
- the **sequential consistency** is a **weaker** model than *linearizability*;
- the **causal consistency** is a **weaker** model than *sequential consistency model*;
- the **eventual consistency** is a **weak** *consistency model*. It defines that if **no** update takes a very long time, **all replicas** *eventually become consistent*;

<br>