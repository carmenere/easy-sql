<!-- TOC -->
* [Architecture of DBMS](#architecture-of-dbms)
* [Column vs. Row vs. Wide column oriented DBMS](#column-vs-row-vs-wide-column-oriented-dbms)
* [Data files and Index files](#data-files-and-index-files)
* [Clustered indexes](#clustered-indexes)
<!-- TOC -->

<br>

# Architecture of DBMS
**DBMS** is abbreviation for **DataBase** **Management** **System**, but sometimes term **database system** or just **database** is used instead.<br>
Databases are modular syatems and consist of multiple components and layers.<br>
The common layers are
- **transport layer** is responsible for **accepting** and **processing connections** from clients and other nodes of cluster;
- **query processor** is responsible for **parsing** and **optimizing** queries and **generating execution plans** (aka **query plans**);
- **execution engine** is responsible for **handling** execution plan and **collecting** the result of **local** and **remote** execution;
- **storage engine** is responsible for executing local queries and contains following components:
  - transaction manager;
  - lock manager;
  - access methods;
  - buffers;
  - recovery subsystem;

<br>

The **high level design** (**HLD**) of typical DBMS id depicted below:<br>
![HLD-DBMS](/img/Architecture_of_DBMS.png)

<br>

The **storage engine** is responsible for **storing**, **retrieving** and **managing** data **in RAM** and **on disk**.<br>
The **DBMS** is a software that **built on the top of** _storage engine_.<br>

Every DBMS has strengths and weaknesses. The **choice** of DBMS may have **long-term consequences**, since it **can be non-trivial to migrate** to a dfifferent DBMS and in some cases, it **may require substantial changes** in the application code.<br>

<br>

The most **common criteria** /kraɪˈtɪərɪə/ of comparison of DBMS are
- size of records;
- number of clients;
- access patterns;
- types of queries;
- rate of read;
- rate of write;

<br>

To choose "right" DBMS for some application the choice must be based on the **workloads** and **use cases**.<br>
And the best thing you can do is to **simulate workloads** against different DBMS, **measure the performance metrics** that are important for you and **compare results**.<br>

<br>

# Column vs. Row vs. Wide column oriented DBMS
**Row oriented DBMSs** store data in **records** (aka **rows** or **tuples** or **heap tuple**).<br>
**Column oriented DBMSs** partition data **vertically** (**by column**), e.g. values for the same column are stored **contiguously**.<br>
**Wide column oriented DBMSs** use hybrid layout: columns are grouped into **column families** and inside each column family data is stored in **rows**.<br>

<br>

# Data files and Index files
**Row oriented DBMSs** store data rows in tables and each **table** is represented as **separate file**.<br>
A DBMS usually separate **data files** and **index files**:
- **data files** (aka **heap files**):
  - store actual **data rows**;
  - rows are **placed in writing order** in heap files;
  - heap files **require** additional **index structures**, pointing to the locations where rows are stored, to make them searchable;
- **index files**:
  - **index files** are organized as specialized structures that map **keys** to appropriate **rows** in **data files**;
  - store **keys** and **pointers** to rows associated with keys;
  - **indexes** allow efficiently locate data without scanning an entire table on every access;

<br>

# Clustered indexes
If the **order of rows** in _heap file_ **follows** the **order of keys** in _index file_ such index is called **clustered index** (aka **clustering index**).<br>
If **order of rows** in heap file **doesn't** follow the **order of keys** in _index file_ such index is called **nonclustered index**.<br>
