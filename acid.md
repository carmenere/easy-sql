# ACID
**ACID** (**atomicity**, **consistency**, **isolation**, **durability**) is a set of properties of database transactions intended to guarantee its reliable work.<br>

**Atomicity** guarantees that each transaction is treated as a single "unit", which either succeeds completely or fails completely: if any of the statements constituting a transaction fails to complete, the entire transaction fails and the database is left unchanged.<br>

**Consistency** ensures that a transaction can only bring the database from one consistent state to another. **Consistency** is achieved with **сonstraints**, **referential integrity**, **triggers**.<br>

**Isolation** ensures that **concurrent** execution of transactions leaves the database in the same state as transactions were executed sequentially. **Isolation** is achieved with **transaction isolation levels**.<br>

**Durability** guarantees that once a transaction has been committed, it will remain committed even in the case of a **system failure** (power outage or crash). **Durability** is achieved with **WAL buffer**.
