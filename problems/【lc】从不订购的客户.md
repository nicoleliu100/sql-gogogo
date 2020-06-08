某网站包含两个表，Customers 表和 Orders 表。编写一个 SQL 查询，找出所有从不订购任何东西的客户。

Customers 表：

+----+-------+
| Id | Name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+
Orders 表：

+----+------------+
| Id | CustomerId |
+----+------------+
| 1  | 3          |
| 2  | 1          |
+----+------------+
例如给定上述表格，你的查询应返回：

+-----------+
| Customers |
+-----------+
| Henry     |
| Max       |
+-----------+

思路

左关联

代码

```
SELECT
	tmp.name AS Customers
FROM
	(
		SELECT
			a.id,
			a.name,
			b.customerId
		FROM
			customers a
		LEFT JOIN orders b
		ON
			a.id = b.CustomerId
	)
	tmp
WHERE
	tmp.CustomerId IS NULL
```

结果

执行结果：

通过

执行用时 :1423 ms, 在所有 mssql 提交中击败了31.78%的用户

内存消耗 :0B, 在所有 mssql 提交中击败了100.00%的用户

