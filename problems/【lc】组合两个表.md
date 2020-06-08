题目

175 组合两个表

描述

SQL架构

表1: `Person`

```
+-------------+---------+
| 列名         | 类型     |
+-------------+---------+
| PersonId    | int     |
| FirstName   | varchar |
| LastName    | varchar |
+-------------+---------+
PersonId 是上表主键
```

表2: `Address`

```
+-------------+---------+
| 列名         | 类型    |
+-------------+---------+
| AddressId   | int     |
| PersonId    | int     |
| City        | varchar |
| State       | varchar |
+-------------+---------+
AddressId 是上表主键
```

编写一个 SQL 查询，满足条件：无论 person 是否有地址信息，都需要基于上述两表提供 person 的以下信息：

```
FirstName, LastName, City, State
```

思路

left join

代码

```
SELECT
	a.FirstName,
	a.LastName,
	b.City,
	b.State
FROM
	(
		SELECT PersonId, FirstName, LastName FROM Person
	)
	a
LEFT JOIN
	(
		SELECT PersonId, City, State FROM Address
	)
	b
ON
	a.PersonId = b.PersonId
```

结果

同样的代码，提交了两次，执行用时差距很大，这。。。。

| 提交时间 | 提交结果 | 执行用时 | 内存消耗 | 语言  |
| :------- | :------- | :------- | :------- | :---- |
| 几秒前   | 通过     | 204 ms   | 0B       | Mysql |
| 几秒前   | 通过     | 376 ms   | 0B       | Mysql |