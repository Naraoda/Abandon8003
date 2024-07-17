# 窗口函数

窗口函数，也叫OLAP函数（Online Anallytical Processing，联机分析处理），可以对数据库数据进行实时分析处理。

语法大致如下

```sql
<窗口函数>over(partition by 分组字段 order by 排序字段)
```

```mysql
窗口函数([参数]) OVER (
      [PARTITION BY <分组列>] 
      [ORDER BY <排序列 ASC/DESC>]
      [ROWS BETWEEN 开始行 AND 结束行]
)
```

主要分为3类，分别是聚合窗口函数、排序窗口函数、偏移窗口函数

1. 聚合窗口函数是avg、sum、count、max、min等；
2. 排序窗口函数是rank、dense_rank、row_number、ntile；
3. 偏移窗口函数是lag、lead

**ROW_NUMBER:**

```
语法：ROW_NUMBER() OVER(PARTITION BY COLUMN ORDER BY COLUMN)
```

简单的说，row_number()从1开始，为每条分组记录返回一个数字，举例：

```
ROW_NUMBER() OVER(ORDER BY xlh DESC)
```

这里的用法是先将xlh列进行降序排序，再将降序后的每条记录返回一个序号。

![img](https://img-blog.csdnimg.cn/fa6060df50b24adb86a6e2cf955f8f53.png)

```
row_number() OVER (PARTITION BY COL1 ORDER BY COL2)
```

表示根据COL1分组，在分组内部根据COL2排序，而此函数计算的值就表示每组内部排序后的顺序编号（组内连续的唯一的）

**lag(expr,N,default)**

- **expr:**它可以是列或任何内置函数。
- **N:**它是一个正值，它确定当前行之前的行数。如果在查询中将其省略，则其默认值为1
- **default:**如果在当前行之前没有行N行的情况下，它是函数返回的默认值。如果缺少，则默认为NULL。

```
select *,lag(start_time,1) over(order by id)lad_1,
lag(uid,2) over(order by id)lad_2 
from exam_record
```

**lead(expr,N,default)**

- expr:它可以是列或任何内置函数。
- N:它是一个正值，它确定当前行之后的行数。如果在查询中将其省略，则其默认值为1
- default:如果在当前行之后没有行N行的情况下，它是函数返回的默认值。如果缺少，则默认为NULL。

```
select *,lead(start_time,1) over(order by id)lead_1,
lead(uid,2) over(order by id)lead_2 
from exam_record
```

[180. 连续出现的数字](https://leetcode.cn/problems/consecutive-numbers/)

```sql
SELECT DISTINCT num AS ConsecutiveNums
FROM (
    SELECT  num, id,
            LEAD(id,1) OVER(ORDER BY id) AS next_id,
            LEAD(num,1) OVER(ORDER BY id) AS next_num,
            LEAD(id,2) OVER(ORDER BY id) AS last_id,
            LEAD(num,2) OVER (ORDER BY id) AS last_num
    FROM Logs 
)AS L
WHERE num=next_num AND num=last_num AND id=next_id-1 AND id=last_id-2
;
```

```
SELECT DISTINCT temp.num AS ConsecutiveNums
FROM 
(  
  SELECT id,num,

  CAST(row_number() OVER(PARTITION BY num ORDER BY id) AS SIGNED) AS rn 

  FROM Logs 

  ORDER BY num ASC,id ASC

) AS temp 

GROUP BY temp.num, temp.id-temp.rn

HAVING COUNT(temp.id-temp.rn)>=3 

;
```

