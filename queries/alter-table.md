# ALTER TABLE
## ADD COLUMN
```sql
ALTER TABLE some_tbl
    ADD COLUMN id BIGSERIAL NOT NULL;
```

<br>

## DROP COLUMN
```sql
ALTER TABLE some_tbl
    DROP COLUMN id;
```

<br>

## ADD CONSTRAINT
### UNIQUE
```sql
ALTER TABLE some_tbl
    ADD CONSTRAINT custom_name_pkey UNIQUE (id);
```

<br>

### CHECK
```sql
ALTER TABLE some_tbl
    ADD CONSTRAINT some_tbl_check
    CHECK (
        (vals = ANY (ARRAY[1, 2, 3, 4, 5]))
    );
```

<br>

### FOREIGN KEY
```sql
ALTER TABLE distributors 
    ADD CONSTRAINT some_fk FOREIGN KEY (col1) REFERENCES tbl2 (col2);
```

<br>

## RENAME CONSTRAINT
```sql
DO $$ BEGIN
    ALTER TABLE child RENAME CONSTRAINT child_forbid_empty_range TO forbid_empty_range;
EXCEPTION
    WHEN undefined_object THEN NULL;
END $$;
```

<br>

## DROP CONSTRAINT
```sql
ALTER TABLE ONLY tbl
    DROP CONSTRAINT tbl_excl;
```

<br>

## ADD PRIMARY KEY
```sql
ALTER TABLE some_tbl 
    ADD PRIMARY KEY (id);
```

<br>

## ALTER COLUMN
### SET NOT NULL
```sql
ALTER TABLE some_tbl 
    ALTER COLUMN my_column SET NOT NULL;
```

<br>

### DROP NOT NULL
```sql
ALTER TABLE ONLY fw_icmp_messages
    ALTER COLUMN code DROP NOT NULL;
```

<br>

### DROP DEFAULT
```sql
ALTER TABLE some_tbl 
    ALTER COLUMN my_column DROP DEFAULT;
```

### Change type of column
PostgreSQL allows you to convert the values of a column to the new ones while changing its data type by adding a `USING` clause as follows:<br>
1. Consider following `enum` types:
```sql
CREATE TYPE echo_reply_t AS ENUM (
    'echo_reply'
);

CREATE TYPE destination_unreachable_t AS ENUM (
    'network_unreachable',
    'host_unreachable',
    'protocol_unreachable',
    'port_unreachable',
    'fragmentation_needed',
    'source_route_failed',
    'network_unknown',
    'host_unknown',
    'source_host_isolated',
    'network_prohibited',
    'host_prohibited',
    'tos_network_unreachable',
    'tos_host_unreachable',
    'communication_prohibited',
    'host_precedence_violation',
    'precedence_cutoff'
);

CREATE TYPE redirect_t AS ENUM (
    'network_redirect',
    'host_redirect',
    'tos_network_redirect',
    'tos_host_redirect'
);

CREATE TYPE echo_request_t AS ENUM (
    'echo_request'
);

CREATE TYPE router_advertisement_t AS ENUM (
    'router_advertisement',
    'does_not_route_common_traffic'
);

CREATE TYPE router_solicitation_t AS ENUM (
    'router_solicitation'
);

CREATE TYPE time_exceeded_t AS ENUM (
    'ttl_zero_during_transit',
    'ttl_zero_during_reassembly'
);

CREATE TYPE parameter_problem_t AS ENUM (
    'ip_header_bad',
    'required_option_missing',
    'bad_length'
);

CREATE TYPE timestamp_request_t AS ENUM (
    'timestamp_request'
);

CREATE TYPE timestamp_reply_t AS ENUM (
    'timestamp_reply'
);
```

<br>

```sql
DO $$ BEGIN
EXECUTE (
    SELECT format('CREATE TYPE icmp_codes_t AS ENUM (%s)', string_agg(DISTINCT quote_literal(variants), ', '))
    FROM (
        SELECT variants FROM UNNEST(
                                    enum_range(NULL::echo_reply_t)::text[]
                                    || enum_range(NULL::destination_unreachable_t)::text[]
                                    || enum_range(NULL::redirect_t)::text[]
                                    || enum_range(NULL::echo_request_t)::text[]
                                    || enum_range(NULL::router_advertisement_t)::text[]
                                    || enum_range(NULL::router_solicitation_t)::text[]
                                    || enum_range(NULL::time_exceeded_t)::text[]
                                    || enum_range(NULL::parameter_problem_t)::text[]
                                    || enum_range(NULL::timestamp_request_t)::text[]
                                    || enum_range(NULL::timestamp_reply_t)::text[]
                                    
                            ) AS variants
        ) t
);
END $$;
```

<br>

Then, convert type of column `code` from `int` to `icmp_codes_t` and change its values to new appropriate values:
```sql
ALTER TABLE ONLY fw_icmp_messages 
    ALTER COLUMN code 
        TYPE icmp_codes_t 
        USING CASE 
            WHEN type = 0  AND code = 0  THEN 'echo_reply'::icmp_codes_t 
            WHEN type = 3  AND code = 0  THEN 'network_unreachable'::icmp_codes_t
            WHEN type = 3  AND code = 1  THEN 'host_unreachable'::icmp_codes_t
            WHEN type = 3  AND code = 2  THEN 'protocol_unreachable'::icmp_codes_t
            WHEN type = 3  AND code = 3  THEN 'port_unreachable'::icmp_codes_t
            WHEN type = 3  AND code = 4  THEN 'fragmentation_needed'::icmp_codes_t
            WHEN type = 3  AND code = 5  THEN 'source_route_failed'::icmp_codes_t
            WHEN type = 3  AND code = 6  THEN 'network_unknown'::icmp_codes_t
            WHEN type = 3  AND code = 7  THEN 'host_unknown'::icmp_codes_t
            WHEN type = 3  AND code = 8  THEN 'source_host_isolated'::icmp_codes_t
            WHEN type = 3  AND code = 9  THEN 'network_prohibited'::icmp_codes_t
            WHEN type = 3  AND code = 10 THEN 'host_prohibited'::icmp_codes_t
            WHEN type = 3  AND code = 11 THEN 'tos_network_unreachable'::icmp_codes_t
            WHEN type = 3  AND code = 12 THEN 'tos_host_unreachable'::icmp_codes_t
            WHEN type = 3  AND code = 13 THEN 'communication_prohibited'::icmp_codes_t
            WHEN type = 3  AND code = 14 THEN 'host_precedence_violation'::icmp_codes_t
            WHEN type = 3  AND code = 15 THEN 'precedence_cutoff'::icmp_codes_t
            WHEN type = 5  AND code = 0  THEN 'network_redirect'::icmp_codes_t
            WHEN type = 5  AND code = 1  THEN 'host_redirect'::icmp_codes_t 
            WHEN type = 5  AND code = 2  THEN 'tos_network_redirect'::icmp_codes_t
            WHEN type = 5  AND code = 3  THEN 'tos_host_redirect'::icmp_codes_t
            WHEN type = 8  AND code = 0  THEN 'echo_request'::icmp_codes_t 
            WHEN type = 9  AND code = 0  THEN 'router_advertisement'::icmp_codes_t
            WHEN type = 9  AND code = 16 THEN 'does_not_route_common_traffic'::icmp_codes_t
            WHEN type = 10 AND code = 0  THEN 'router_solicitation'::icmp_codes_t
            WHEN type = 11 AND code = 0  THEN 'ttl_zero_during_transit'::icmp_codes_t
            WHEN type = 11 AND code = 1  THEN 'ttl_zero_during_reassembly'::icmp_codes_t
            WHEN type = 12 AND code = 0  THEN 'ip_header_bad'::icmp_codes_t
            WHEN type = 12 AND code = 1  THEN 'required_option_missing'::icmp_codes_t
            WHEN type = 12 AND code = 2  THEN 'bad_length'::icmp_codes_t
            WHEN type = 13 AND code = 0  THEN 'timestamp_request'::icmp_codes_t
            WHEN type = 14 AND code = 0  THEN 'timestamp_reply'::icmp_codes_t
            ELSE NULL
        END;
```