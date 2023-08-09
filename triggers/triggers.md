# Triggers
## Example
```sql
CREATE TRIGGER my_trigger
  AFTER INSERT
  ON tbl
  FOR EACH ROW
  EXECUTE PROCEDURE set_tstz();
```

<br>

https://www.postgresql.org/docs/current/sql-createtrigger.html
