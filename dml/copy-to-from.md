# COPY .. TO ../COPY .. FROM ..
```sql
COPY table_name [ ( col_1 [, ...] ) ] FROM { 'path_to_file' | STDIN }
[ [ WITH ] ( option [, ...] ) ]
```

```sql
COPY table_name [ ( col_1 [, ...] ) ] TO { 'path_to_file' | STDOUT }
[ [ WITH ] ( option [, ...] ) ]
```

**where**
- `STDIN` specifies that **input** comes **from** the **client application**;
- `STDOUT` specifies that **output** goes **to** the **client application**;
- `FORMAT` specifies the data format to be read or written:
  - `text`
  - `csv`
  - `binary` it is possible to insert data in binary format to table directly through `COPY table_name FROM STDIN ...`
- `option` can be one of:
    - `FORMAT format_name`
    - `DELIMITER 'delimiter_character'`
    - `NULL 'null_string'` specifies the string that represents a null value.
    - `DEFAULT 'default_string'`
    - `HEADER [ boolean | MATCH ]`
      - **on output**, the **first line** contains the column names from the table will be written to file;
      - **on input**, the **first line** is **discarded** when this option is **set** to `true`: `HEADER true`;
      - `HEADER MATCH` requires the number and names of the columns in the **header line** must match the actual column names of the table, in order, otherwise an error is raised;
    - ...

<br>

# Load data FROM CSV file TO the table
```sql
COPY table_name FROM '/tmp/table_name.csv' WITH (FORMAT csv);
```

<br>

# Export data FROM table TO CSV file
```sql
COPY table_name TO '/tmp/table_name.csv' DELIMITER ',' CSV HEADER;
```

<br>

# COPY .. FROM STDIN as INSERT
```sql
COPY foo FROM STDIN WITH (FORMAT csv);
Enter data to be copied followed by a newline.
End with a backslash and a period on a line by itself, or an EOF signal.
>> 33,abc
>> 44,xyz
>> \.
COPY 2
```