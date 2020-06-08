题目

第二高的薪水

描述

SQL架构

编写一个 SQL 查询，获取 `Employee` 表中第二高的薪水（Salary） 。

```
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
```

例如上述 `Employee` 表，SQL查询应该返回 `200` 作为第二高的薪水。如果不存在第二高的薪水，那么查询应返回 `null`。

```
+---------------------+
| SecondHighestSalary |
+---------------------+
| 200                 |
+---------------------+
```

思路

- 方法1：窗口函数  row_number 或者 rank等
- 方法2：max()取最大薪水，比max小的第一个数就是第二大的



代码

```
SELECT
	Salary
FROM
	(
		SELECT Salary, rank()over(order by Salary DESC) AS rank FROM Employee
	)tmp
WHERE
	rank = 2
```



```
SELECT
	Salary
FROM
	Employee
ORDER BY
	Salary DESC
WHERE
	Salary <
	(
		SELECT MAX(Salary) FROM Employee
	)
	limit 1
```



