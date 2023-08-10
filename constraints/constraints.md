# Constraints
**Column** level:
- `DEFAULT value`;
- `NOT NULL` or `NULL`;
- `UNIQUE` (per column);

<br>

**Table** level:
- `PRIMARY KEY`;
- `FOREIGN KEY`;
- `INDEX`;
- `UNIQUE`
- `CHECK`;
- `EXCLUDE`;

<br>

# Errors
## PostgreSQL error codes
**Error codes** are specified in the **SQL-92 standard**.
[PostgreSQL Error Codes](https://www.postgresql.org/docs/current/errcodes-appendix.html)

<br>

## Class 23 — Integrity constraint violation
|Error Code|Condition Name|
|:---------|:-------------|
|23000|**integrity_constraint_violation**|
|23001|**restrict_violation**|
|23502|**not_null_violation**|
|23503|**foreign_key_violation**|
|23505|**unique_violation**|
|23514|**check_violation**|
|23P01|**exclusion_violation**|

<br>

> **Note**:<br>
> **Not-null constraint** hasn't **name** and field "**constraint**" in **error message** from pg is filled by `NONE`.

<br>

```sh
23514   check_violation:
            table: customers
            column: NONE
            data_type: NONE
            constraint: customers_name_check
            code: 23514
            message: new row for relation "customers" violates check constraint "customers_name_check"
            detail: Failing row contains (43091324936, ).
            hint: NONE
            where: NONE
            schema: public
```

<br>

```sh
23502   not_null_violation:
            table: customers
            column: id
            data_type: NONE
            constraint: NONE
            code: 23502
            message: null value in column "id" violates not-null constraint
            detail: Failing row contains (null, null).
            hint: NONE
            where: NONE
            schema: public
```


