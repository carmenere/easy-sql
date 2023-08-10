# Deferrable constraint
## Set deferrable constraint in table
```sql
...
FOREIGN KEY (pid) REFERENCES parent(id) ON DELETE CASCADE DEFERRABLE INITIALLY DEFERRED,
UNIQUE (pid, name) DEFERRABLE INITIALLY DEFERRED,
...
```

<br>

## Alter deferrable constraint in table
```sql
ALTER TABLE tbl1 ALTER CONSTRAINT fk1 DEFERRABLE INITIALLY DEFERRED;
```