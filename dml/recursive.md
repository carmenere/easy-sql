# Recursive queries
A **recursive query** is a query that refers to a **recursive CTE**.<br>

```sql
WITH RECURSIVE t AS (
    /* Query 1 */ -- non-recursive term
    UNION [ALL]
    /* Query 2 */  -- recursive term
)
SELECT * FROM t;
```

<br>

A **recursive CTE** has three elements:
- **non-recursive term** is a query that forms the **base result set** of the CTE structure;
- **recursive term** is a query that is joined with the **non-recursive term** using the `UNION` or `UNION ALL` operator;
- **termination condition**: the **recursion stops** when **no** rows are returned from the **previous** iteration;

<br>

Flow:
1. **Non-recursive term** (executed only once at the beginning):
- reads original **table**;
- all resulting rows of **non-recursive term** are put into **result set** and into **temporary set**;

2. **Recursive term** (executed repeatedly):
- **temporary set** is **empty** at the beginning;
- reads original **table** and also has access to **temporary set**;
- all resulting rows of **non-recursive term** are put into **result set** and into **current set**;

3. Checking **termination condition**:
- if **current set** is **not** empty all its rows are moved to **temporary set** and step 2 is repeated;
- if **current set** is empty the process stops;

<br>

## Example
1. **Preparations**
```sql
CREATE TABLE IF NOT EXISTS commands (
    id INTEGER PRIMARY KEY,
    command TEXT NOT NULL,
    parent INTEGER,
    FOREIGN KEY(parent) REFERENCES commands(id)
) STRICT;

CREATE UNIQUE INDEX commands_idx1 ON commands(command) WHERE parent IS NULL;
CREATE UNIQUE INDEX commands_idx2 ON commands(command,parent) WHERE parent IS NOT NULL;

INSERT INTO commands(command)
VALUES
    ('docker'),
    ('git'),
    ('brew'),
    ('psql')
;

INSERT INTO commands(parent, command)
VALUES
    ((SELECT id FROM commands WHERE command='docker'), 'build'),
    ((SELECT id FROM commands WHERE command='docker'), 'check'),
    ((SELECT id FROM commands WHERE command='docker'), 'exec_i_cmd'),
    ((SELECT id FROM commands WHERE command='docker'), 'exec_it_cmd'),
    ((SELECT id FROM commands WHERE command='docker'), 'exec_sh'),
    ((SELECT id FROM commands WHERE command='docker'), 'logs'),
    ((SELECT id FROM commands WHERE command='docker'), 'network_create'),
    ((SELECT id FROM commands WHERE command='docker'), 'network_ls'),
    ((SELECT id FROM commands WHERE command='docker'), 'network_rm'),
    ((SELECT id FROM commands WHERE command='docker'), 'pull'),
    ((SELECT id FROM commands WHERE command='docker'), 'rm'),
    ((SELECT id FROM commands WHERE command='docker'), 'rmi'),
    ((SELECT id FROM commands WHERE command='docker'), 'run'),
    ((SELECT id FROM commands WHERE command='docker'), 'start'),
    ((SELECT id FROM commands WHERE command='docker'), 'status'),
    ((SELECT id FROM commands WHERE command='docker'), 'stop'),

    ((SELECT id FROM commands WHERE command='psql'), 'alter_role_password'),
    ((SELECT id FROM commands WHERE command='psql'), 'conn'),
    ((SELECT id FROM commands WHERE command='psql'), 'conn_local'),
    ((SELECT id FROM commands WHERE command='psql'), 'create_db'),
    ((SELECT id FROM commands WHERE command='psql'), 'create_user'),
    ((SELECT id FROM commands WHERE command='psql'), 'drop_db'),
    ((SELECT id FROM commands WHERE command='psql'), 'drop_role_password'),
    ((SELECT id FROM commands WHERE command='psql'), 'drop_user'),
    ((SELECT id FROM commands WHERE command='psql'), 'grant_user'),
    ((SELECT id FROM commands WHERE command='psql'), 'revoke_user'),

    ((SELECT id FROM commands WHERE command='brew'), 'services_list'),
    ((SELECT id FROM commands WHERE command='brew'), 'services_restart'),
    ((SELECT id FROM commands WHERE command='brew'), 'services_start'),
    ((SELECT id FROM commands WHERE command='brew'), 'services_stop')
;

sqlite> select * from commands;
id  command              parent
--  -------------------  ------
1   docker               <NULL>
2   git                  <NULL>
3   brew                 <NULL>
4   psql                 <NULL>
5   build                1
6   check                1
7   exec_i_cmd           1
8   exec_it_cmd          1
9   exec_sh              1
10  logs                 1
11  network_create       1
12  network_ls           1
13  network_rm           1
14  pull                 1
15  rm                   1
16  rmi                  1
17  run                  1
18  start                1
19  status               1
20  stop                 1
21  alter_role_password  4
22  conn                 4
23  conn_local           4
24  create_db            4
25  create_user          4
26  drop_db              4
27  drop_role_password   4
28  drop_user            4
29  grant_user           4
30  revoke_user          4
31  services_list        3
32  services_restart     3
33  services_start       3
34  services_stop        3
sqlite>
```

2. **Recursive query**
```sql
WITH RECURSIVE r(id, command, parent, full_command, is_leaf) AS (
    SELECT c.id,
           c.command,
           c.parent,
           c.command AS full_command,
           NOT EXISTS (SELECT 1
                       FROM commands cc
                       WHERE c.id = cc.parent) AS is_leaf
    FROM commands AS c
    WHERE c.parent IS NULL
    UNION
    SELECT c.id,
           c.command,
           c.parent,
           concat(r.full_command, '_', c.command) AS full_command,
           NOT EXISTS (SELECT NULL
                       FROM commands cc
                       WHERE c.id = cc.parent) AS is_leaf
    FROM commands AS c JOIN r ON r.id = c.parent
)
SELECT *
FROM r
WHERE r.is_leaf = 1;
                         
id  command              parent  full_command              is_leaf
--  -------------------  ------  ------------------------  -------
2   git                  <NULL>  git                       1
31  services_list        3       brew_services_list        1
32  services_restart     3       brew_services_restart     1
33  services_start       3       brew_services_start       1
34  services_stop        3       brew_services_stop        1
5   build                1       docker_build              1
6   check                1       docker_check              1
7   exec_i_cmd           1       docker_exec_i_cmd         1
8   exec_it_cmd          1       docker_exec_it_cmd        1
9   exec_sh              1       docker_exec_sh            1
10  logs                 1       docker_logs               1
11  network_create       1       docker_network_create     1
12  network_ls           1       docker_network_ls         1
13  network_rm           1       docker_network_rm         1
14  pull                 1       docker_pull               1
15  rm                   1       docker_rm                 1
16  rmi                  1       docker_rmi                1
17  run                  1       docker_run                1
18  start                1       docker_start              1
19  status               1       docker_status             1
20  stop                 1       docker_stop               1
21  alter_role_password  4       psql_alter_role_password  1
22  conn                 4       psql_conn                 1
23  conn_local           4       psql_conn_local           1
24  create_db            4       psql_create_db            1
25  create_user          4       psql_create_user          1
26  drop_db              4       psql_drop_db              1
27  drop_role_password   4       psql_drop_role_password   1
28  drop_user            4       psql_drop_user            1
29  grant_user           4       psql_grant_user           1
30  revoke_user          4       psql_revoke_user          1
sqlite>
```