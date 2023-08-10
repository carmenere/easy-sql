# Exclude constraint
1. Activate **extension** `btree_gist`:
```sql
CREATE EXTENSION btree_gist;
CREATE EXTENSION
```
2. Use **exclude** constraint `EXCLUDE USING ... `:
```sql
CREATE TABLE bookings (
    vehicle_id SERIAL PRIMARY KEY,
    travel_dates tstzrange NOT NULL,
    EXCLUDE USING gist (vehicle_id WITH =, travel_dates WITH &&)
);
```

<br>

```sql
CREATE TABLE my_table
(                                  
    id BIGINT NOT NULL,
    name VARCHAR(15) NOT NULL,
    prefix inet NOT NULL,
    EXCLUDE USING gist (id WITH =, prefix inet_ops WITH &&)
);
```
Here `inet_ops`	is an **operator class**.