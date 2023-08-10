# ORDER BY
There is `ORDER BY` to sort rows.<br>
It is possible to set *sort direction* **per** column:
- `ASC` (ascending) means `from 0 to N` or `from A to Z`;
- `DESC` (descending) means `from N to 0` or `from Z to A`;

<br>

### Example
```sql
SELECT id, price, name
FROM (VALUES (1, 100, 'foo1'), (1, 200, 'foo2'), (1, 110, 'bar1'), (1, 200, 'bar2')) AS t(id, price, name)
ORDER BY price DESC, name ASC;

 id | price | name
----+-------+------
  1 |   200 | bar2
  1 |   200 | foo2
  1 |   110 | bar1
  1 |   100 | foo1
(4 rows)
```
