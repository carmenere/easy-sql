# ALTER TABLE
## ADD COLUMN
```sql
ALTER TABLE some_tbl
    ADD COLUMN id BIGSERIAL NOT NULL;
```

<br>

## ADD CONSTRAINT
```sql
ALTER TABLE some_tbl
    ADD CONSTRAINT custom_name_pkey UNIQUE (id);
```

<br>

```sql
ALTER TABLE distributors 
    ADD CONSTRAINT some_fk FOREIGN KEY (col1) REFERENCES tbl2 (col2);
```

<br>

## ADD PRIMARY KEY
```sql
ALTER TABLE some_tbl 
    ADD PRIMARY KEY (id);
```

<br>

## ALTER COLUMN
```sql
ALTER TABLE some_tbl 
    ALTER COLUMN my_column SET NOT NULL;
```

<br>

```sql
ALTER TABLE some_tbl 
    ALTER COLUMN my_column DROP DEFAULT;
```
