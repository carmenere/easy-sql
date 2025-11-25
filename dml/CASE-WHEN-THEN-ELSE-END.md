# CASE-WHEN-THEN-ELSE-END
```sql
demo=# SELECT model, range AS distance,
    CASE
        WHEN range < 2000 THEN 'Ближнемагистральный'
        WHEN range < 5000 THEN 'Среднемагистральный'
        ELSE 'Дальнемагистральный'
    END AS class
    FROM airplanes ORDER BY range;

          model          | distance |        class
-------------------------+----------+---------------------
 Bombardier CRJ700       |     3100 | Среднемагистральный
 Embraer E170            |     4000 | Среднемагистральный
 Boeing 767-300F         |     6000 | Дальнемагистральный
 Aerobus A320neo         |     6500 | Дальнемагистральный
 Boeing 737 MAX 7        |     7000 | Дальнемагистральный
 Aerobus A350F           |     8700 | Дальнемагистральный
 Aerobus A330-900neo     |    13300 | Дальнемагистральный
 Boeing 787-9 Dreamliner |    14000 | Дальнемагистральный
 Boeing 777-300ER        |    14600 | Дальнемагистральный
 Aerobus A350-1000       |    16700 | Дальнемагистральный
(10 rows)

```