# Cluster
A **database cluster** is a **collection** of **databases** (**catalogs**) that are managed by a **single server instance**.<br>

The word **cluster** is defined by the **SQL standard**, e.g., in **SQL-92** specification:
- a **cluster** is an implementation-defined **collection of catalogs**;
- exactly **one** cluster is associated with an **SQL session**.

<br>

Each **cluster** is **listening** on its **own assigned port** for incoming database connections.<br>

<br>

Most of DB provides this full hierarchy: `Cluster > Catalog > Schema > Table`.

So,
- **single server instance** is a **cluster**.
- *cluster* has **catalogs**. (**catalog** = **database**)
- *catalogs* have **schemas**. (**schema** is a **namespace** for tables)
- *schemas* have **tables**.
- *tables* have **rows**, **columns** and **cells** (**values**).

<br>

**Fully qualified name** of **table** inside cluster: `catalog.schema.table`. 
