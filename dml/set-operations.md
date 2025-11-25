# Combining queries
The results of two queries can be **combined** using the **set operations**.<br>

The syntax is
- **set union**: `query_1 UNION [ALL] query_2`;
- **set intersection**: `query_1 INTERSECT [ALL] query_2`;
- **set difference**: `query_1 EXCEPT [ALL] query_2`;

> **Notes**:<br>
> **With** `ALL` keyword duplicate rows are **kept** in result.<br>
> **Without** `ALL` keyword duplicate rows are **eliminated** from result.<br>

<br>

It is possible to use paranthesis:
```sql
(
    SELECT departure_airport, arrival_airport FROM routes WHERE days_of_week @> '{2,3,4}'::int4[]
)
EXCEPT
(
    SELECT departure_airport, arrival_airport FROM routes WHERE days_of_week @> '{6,7}'::int4[]
);

 departure_airport | arrival_airport
-------------------+-----------------
 ATL               | YYZ
 LKO               | KTM
 BER               | ARN
 SZX               | PVG
 KTM               | KMG
 HND               | PEK
 LAS               | ORD
 ARN               | BER
 KTM               | LKO
 ORY               | FRA
 ORY               | VIE
 MXP               | LGW
 PEK               | SZX
 PNQ               | HYD
 TFU               | HGH
 LED               | OVB
 FRA               | MXP
 OVB               | LED
 KTM               | CCU
 VIE               | ORY
 HYD               | CCU
(21 rows)
```