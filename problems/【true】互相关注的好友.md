

描述：有一张表有两个字段，第一个字段是用户，第二个字段是该用户关注了的好友，求互相关注的用户。

假设有这么几行记录：

| user | followed |
| ---- | -------- |
| A    | B        |
| A    | C        |
| A    | D        |
| A    | E        |
| B    | A        |
| D    | A        |

根据定义，应该是下面圈出来的几条记录是互相关注的好友：

![image-20200415192050068](D:\nicole-github\sql-gogogo\pics\image-20200415192050068.png)



因为只有一张表，很自然的想到应该是使用自关联，自己join自己。

仔细观察，join的条件应该是两个：

a.user=b.followed and a.followed=b.user

```
SELECT
	a.user,
	a.followed_user
FROM
	(
		SELECT USER, followed_user FROM test_table
	)a
JOIN
	(
		SELECT USER, followed_user FROM test_table
	)b
ON
	a.user = b.followed_user
	AND a.followed_user = b.user
```

