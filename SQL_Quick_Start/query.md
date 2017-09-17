# Query

- `select`, `from`, `where`

```{sql}
select table1.attribute1 as attribute1_new_name, table2_new_name.attribute2, table3.attribute3, table4.attribute4, table5.attribute5
from table1, table2 as table2_new_name, (table 3 natural join table 4) join table 5 using (attribute_ID)
where condition1, condition2
```
Note:
  - 笛卡尔乘积
  - natural join: 只考虑两个表在所有共同属性的取值相同
  - join table 


- distinct
```
select distinct
```

- order by, desc, asc

- and or not

- null

