# Table of contents
<!-- TOC -->
- [Table of contents](#table-of-contents)
- [Integrity vs. Consistency](#integrity-vs-consistency)
- [Coherence vs. Consistency](#coherence-vs-consistency)
- [Consistency](#consistency)
  - [Consistency models](#consistency-models)
  - [Linearizability](#linearizability)
  - [CAP Theorem](#cap-theorem)
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

<br>

All *consistency models* **under** *strong serializability* are divided into **2 groups**:
- **multi-object** *consistency models*:
  - **read uncommitted**;
  - **read committed**;
  - **repeatable read**;
  - **serializability**;
- **single-object** *consistency models*:
  - **linearizability** (aka **strong consistency**);
    - *every operation* takes place **atomically**, **in some order**, according the **real-time ordering**;
  - **sequential consistency**, it is a **weaker** than *linearizability*;
    - it is a **relaxed model** compared to linearizability because it **doesn't care** about the **real time** of the events while ordering them;
  - **causal consistency**, it is a **weaker** than *sequential consistency*;
    - it requires that **casually related writes** must be seen in the **same order** by **all nodes** in the distributed system;
  - **eventual consistency**, it is a **weak** *consistency model*;
    - it defines that if **no** update takes a very long time, **all replicas** *eventually become consistent*;

<br>

**Strong serializability** isthe **strongest** *consistency model*, it combines **serializability** and **linearizability**. **Linearizability** is often confused with **serializability** - another consistency model.<br>

<br>

**Linearizability** vs. **Serializability**:
- **Linearizability** is one of the **strongest single-object** *consistency model*;
  - in other words, *linearizability* is a **guarantee** about *single operations* on *single objects*;
- **Serializability** is one of the **strongest multi-object** *consistency model*;
  - in other words, *serializability* is a **guarantee** about *transactions* (*groups of one or more operations*) over *multiple objects*;

<br>

So, **serializability** is an **isolation property** of transactions, which **guarantees** that even though transactions may execute *concurrently* over *multiple objects*, the **end result** is the **same as** if they had executed **serially**, i.e. *sequentially*, *one after another*, **without** *any concurrency*. *Serializability* refers to the **I** or **isolation**, in **ACID**.<br>

<br>

## Linearizability
**Linearizability** implies that:
- *every operation* takes place **atomically**, **in some order**, according the **real-time ordering** (as operations appeared according **real-time clock**);
  - if *read* operation returns **version 2** of a data (e.g., `x=2`), **all** subsequent *read* operations **must** also return *version >= 2*: (`x=2`) **or** *updates happened afterward*;
- **result** of each operation must be **propogated instantly** accross entire *distributed system*;
  - if *write* operation `A` completes **before** *read* operation `B` begins, then `B` **must** see the result of the `A`, in other words *write* must be propogated **instantly** accross all nodes in *distributed system*;

<br>

In a *linearizable system* **all operations** must happen **atomically**  **Linearizability** makes *distributed system* behave like a **single**, **atomic**, **non-distributed** *system*.<br>

In a *linearizable system* every client always sees the **most recent up-to-date value**.<br>

<br>

**Thread-safety** *implies linearizability*.<br>

<br>

Common **approaches** to achieve linearizability:
- **single-leader replication**;
  - **all write operations** go to a single, designated **leader node**;
- **quorum-based consistency** (e.g., *Paxos*, *Raft*);
  - operations require **agreement from a majority** (**quorum**) of nodes before being considered complete;
    - for a **write**, a **quorum** of nodes must acknowledge the write;
    - for a **read**, a **quorum** of nodes must be queried to ensure the latest data is retrieved;
- **atomic broadcast** (e.g., *Zookeeper’s ZAB*, *Apache Kafka’s Raft*);

<br>

## CAP Theorem
The **CAP theorem** states that a distributed data store cannot simultaneously provide more than two out of the following three guarantees:
- **Consistency** (**C**): every request receives the **most recent write** or **an error**;
- **Availability** (**A**): every request receives a (*non-error*) **response**, **without** the **guarantee** that it contains the *most recent write*;
- **Partition tolerance** (**P**): the system **continues to operate** despite arbitrary *network failures* (*partitions*) that cause some messages to be dropped or delayed;

<br>

*In practice*, for any interesting distributed system, **P** **is given**. **Network failures will happen**. This means you **must choose between** **C** and **A** **during a network partition**:
- **linearizable systems** (aka **CP**) **prioritize consistency**. If a **network partition** occurs, the system **might become unavailable to some clients** to ensure all remaining available nodes have a consistent view of the data;
  - examples: *etcd*, *ZooKeeper*, *distributed databases using Paxos/Raft*;
- **eventually consistent systems** (aka **AP**): **prioritize availability**. During a **network partition**, the **system remains available**, but **different parts** of the system might have **inconsistent** views of the data. **Consistency** is **eventually achieved** once the partition heals;
  - examples: *Cassandra*, *DynamoDB*;

<br>
