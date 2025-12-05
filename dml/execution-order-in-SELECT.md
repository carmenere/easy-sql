# Execution order in SELECT
1. `FROM`: **gets all records** from table or joined tables
2. `WHERE`: **filters** records
3. `GROUP BY`: **aggregates**
4. `HAVING`: **filters** groups
5. `SELECT`: takes **specific columns**
6. `func() OVER ()`: applies **windows functions**
7. `ORDER BY`: **sorts** the result
8. `LIMIT n OFFSET m`
