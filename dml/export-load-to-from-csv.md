# Load data FROM CSV file TO the table
```sql
COPY table_name FROM '/tmp/table_name.csv' WITH (FORMAT csv);
```

<br>

# Export data FROM table TO CSV file
```sql
COPY table_name TO '/tmp/table_name.csv' DELIMITER ',' CSV HEADER;
```
