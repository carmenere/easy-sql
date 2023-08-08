# Roles
**Role** is an entity that *can own* **database objects** and *have* **attributes** and **privileges** on database objects.<br>
PostgreSQL doesn't have **separate** entities to represent *users* and *groups*. **Any role** can act as a *user*, a *group*, or *both*.<br>

## Show roles
1. By using SQL:
```sql
SELECT * FROM pg_roles WHERE rolname='postgres';

SELECT * FROM pg_roles;
```
2. By using `\du` command:
```sql
my_db=# \du
                                     List of roles
   Role name   |                         Attributes                         | Member of
---------------+------------------------------------------------------------+-----------
 postgres      | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 rust_analyzer | Superuser, Create DB                                       | {}
 foo           | Superuser, Create DB                                       | {}
```

<br>

# Ownership
**Owner** is a role that is configured as the owner for particular object of database: *whole databas*e, *table*, and so on.<br>
**Ownership** gives to role **full access** to the **owned object**.<br>

> **Note**<br>
> By default, role owns any objects that was created by that role;<br>
> Each *database* and *table* have **exactly one owner**;<br>
> **Owner** can **modify** or **delete** objects it owns;<br>
> **Superuser** can **modify** or **delete** objects that it even **doesn't own**;

<br>

An object can be **reassigned** to a **new owner** with an `ALTER` command of the **appropriate kind**:
```sql
ALTER TABLE table_name OWNER TO new_owner;
```

<br>

## Role attributes
**Role attributes** are **role's global capabilities**.<br>
**Role attributes** can be **set** when the role is initially created and **changed** *at any time* by any role with the appropriate privilege (`SUPERUSER` or `CREATEROLE`).<br>
Role attributes:
|Attribute|Description|
|:--------|:----------|
|`LOGIN`|Allows role to be used for following connection types: **host**, **hostssl**, **hostnossl**.|
|`SUPERUSER`|Allows the role to **bypass all permission checks** except the right to log in. It is the **most powerful attribute**.<br>There must always be **at least one** role with superuser attribute in each database cluster.<br>The **initial superuser** account is created during the installation process.<br>The name of the initial superuser account can vary, but most often, this account is called **postgres**.|
|`CREATEDB`|Allows the role to **create new databases**.|
|`CREATEROLE`|Allows the role to **create**, **alter**, and **delete** other roles.|
|`PASSWORD 'text'`<br>`ENCRYPTED PASSWORD 'text'`|Assigns a password to the role that will be used with `password` or `md5` **auth methods**.|
|`INHERIT`|This attribute is set for new roles by default. Determines whether the role inherits the privileges of roles it is a member of.<br>Without the `INHERIT`, members must use `SET ROLE` to change into the other role in order to access those exclusive privileges.|
|`REPLICATION`|Allows the role to initiate **streaming replication** of cluster. Roles with this attribute must also have the LOGIN attribute.|

<br>

## Commands
**Roles** are managed with the following commands:
1. Create:
```sql
CREATE ROLE role_name WITH attribute1 attribute2 ... ;
```
2. Change:
```sql
ALTER ROLE role_name WITH attribute1 attribute2 ... ;
```
3. Delete:
```sql
DROP ROLE role_name;
```

<br>

`CREATE USER` is an **alias** for `CREATE ROLE`. The `CREATE USER` command **automatically adds** attribute `LOGIN` to role, while `CREATE ROLE` command **doesn't**.

<br>

So, command:
```sql
CREATE USER foo WITH ENCRYPTED PASSWORD '1';
```
**is equal to command**:
```sql
CREATE ROLE foo WITH LOGIN ENCRYPTED PASSWORD '1';
```

<br>

## Group roles
A **group** is a **role** that will be used **as group**.<br>
To set up a **group role**, first create some role:
```sql
CREATE ROLE my_group;
```

<br>

By convention, a **group role** does not have the `LOGIN` privilege. It means that you will not be able to use the **group role** to **log in** to PostgreSQL.<br>

Once the **group role** exists, you can **grant membership** in a role `my_group` to another roles or **remove members** from `my_group`:
```sql
GRANT my_group TO role1, ... ;
REVOKE my_group FROM role1, ... ;
```

<br>

The members of a group role can use the privileges of the role in two ways:
1. Every member of a group can explicitly do `SET ROLE role_name`. In this state, the database session has access to the privileges of the role `role_name` rather than the original login role, and any database objects created are considered owned by the role `role_name` not the login role.
2. Roles that have the `INHERIT` attribute automatically **inherit** all privileges of roles of which they are members, **including any** privileges **inherited** by those roles.

<br>

### Example
```sql
CREATE ROLE joe LOGIN INHERIT;
CREATE ROLE admin NOINHERIT;
CREATE ROLE wheel NOINHERIT;
GRANT admin TO joe;
GRANT wheel TO admin;
```

<br>

`joe` has privileges granted **directly** to `joe` **plus** any privileges granted to `admin`, because `joe` **inherits** `admin`'s privileges.<br>
However, privileges granted to `wheel` are **not available**, because even though `joe` is **indirectly** a member of `wheel`, the `admin` has the `NOINHERIT` attribute and **doesn't** automatically inherit `wheel`'s privileges.<br>
So, to get `wheel`'s privileges following commands must be run:
```sql
SET ROLE admin;
SET ROLE wheel;
```

<br>

## Privileges
For most kinds of objects, the initial state is that **only the owner** (or a **superuser**) can **do anything** with the object. To **allow other** roles to use it, **privileges** must be **granted**.<br>

**Privileges** on database objects are managed with the `GRANT` and `REVOKE` commands:
- to **assign** privileges, the `GRANT privilege [, ...] ON object [, ...] TO { PUBLIC | role }` command is used;
- to **revoke** a previously-granted privilege, the `REVOKE privilege ON object FROM role` command is used;

<br>

> **Note**:<br>
> The special role name `PUBLIC` can be used to grant a privilege to **every role** on the system.<br>
> Writing `ALL` in place of a specific privilege **grants all privileges** that are relevant for the object type.

<br>

There are different kinds of privileges: 
- `SELECT`
- `INSERT`
- `UPDATE`
- `DELETE`
- `TRUNCATE`
- `REFERENCES`
- `TRIGGER`
- `CREATE`
- `CONNECT`
- `TEMPORARY`
- `EXECUTE`
- `USAGE`
- `SET`
- `ALTER SYSTEM`

<br>

### Examples
```sql
mydb=# GRANT SELECT ON some_tbl TO some_user;
GRANT

mydb=# REVOKE SELECT ON some_tbl FROM some_user;
REVOKE
```

<br>

# Managing roles and privileges
## Create role
1. Create user
```sql
CREATE USER foo WITH ENCRYPTED PASSWORD 'text';
```
2. Create db
```sql
CREATE DATABASE bar;
```
3. Grant priviliges
```sql
GRANT ALL PRIVILEGES ON DATABASE bar TO foo;
```

<br>

## Change role
```sql
ALTER ROLE foo WITH LOGIN CREATEROLE CREATEDB SUPERUSER REPLICATION BYPASSRLS;
```

Remove password:
```sql
ALTER ROLE foo PASSWORD NULL;
```

<br>

## Delete role
Role **cannot** be removed if it is **referenced** by **at least one** object within the database.<br>

Steps to delete role `foo`:
1. **Reassign** or **Delete**.
- **Reassign** all role’s owned objects to another role:
```sql
ALTER DATABASE bar OWNER TO postgres;
REASSIGN OWNED BY foo TO postgres;
```
- **Delete** all objects owned by role, (if ownership was transferred to another role, e.g. `postgres`, the `foo` role no longer has any owned objects):
```sql
DROP OWNED BY foo;
```
1. **Delete role**:
```sql
DROP ROLE foo;
```

<br>

## Database owner
1.	Assign owner to db **while** db is creating:
```sql
CREATE DATABASE my_db WITH OWNER some_user;
```
2.	Assign owner to db **after** db was created:
```sql
ALTER DATABASE my_db OWNER TO some_user;
```

