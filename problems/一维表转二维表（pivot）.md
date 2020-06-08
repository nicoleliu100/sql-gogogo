下面是学生的成绩表(表名：成绩表，列名：学号，课程，成绩)

| student_id | course | score |
| ---------- | ------ | ----- |
| 1          | 语文   | 80    |
| 1          | 数学   | 75    |
| 2          | 语文   | 95    |
| 2          | 数学   | 98    |
| 3          | 语文   | 79    |
| ...        | ...    | ...   |

要求使用sql输出如下形式的二维表格：

| student_id | chinese_score | math_score |
| ---------- | ------------- | ---------- |
| 1          | 80            | 75         |
| 2          | 95            | 98         |
| 3          | 79            | 90         |



思路

原表是普通的一维表格，所谓一维是指只有column代表的含义是唯一的，而要求输出的表格是二维的，student_id和横向的学科成绩组成唯一的pair确定一个值（分数）。

是不是有点像excel里面创建pivot table，选择行和列，然后生成一个二维的透视表。

这类题目可以抽象出统一的解答步骤：

1. 先使用case when把每一行的列进行扩展，使形式上和最后期望的输出列一致，行数依然和原表格相同（没有使用到group by 等聚合操作）。代码如下：

```
SELECT
	student_id,
	CASE
		WHEN course = '语文'
		THEN score
		ELSE 0
	END AS chinese_score,
	CASE
		WHEN course = '数学'
		THEN score
		ELSE 0
	END AS math_score
FROM
	table1select
```

这样得到的结果为：

| student_id | chinese_score | math_score |
| ---------- | ------------- | ---------- |
| 1          | 80            | 0          |
| 1          | 0             | 75         |
| 2          | 95            | 0          |
| 2          | 0             | 98         |

2. 显然最后的期望输出是每个student_id只有一行结果，但是上面的结果每个学生对应有两行结果，因此需要对上述结果进行聚合操作。我们对student_id进行group by， 用student_id=1的学生来举例，可以知道其chinese_score组成的列表为[80,0], math_score组成的列表为[0,75], 我们只需要取每列列表的最大值即可。所以这里使用的是max()函数来进行聚合。代码如下：

```
SELECT
	student_id,
	MAX(chinese_score) AS chinese_score,
	MAX(math_score) AS math_score
FROM
	(
		SELECT
			student_id,
			CASE
				WHEN course = '语文'
				THEN score
				ELSE 0
			END AS chinese_score,
			CASE
				WHEN course = '数学'
				THEN score
				ELSE 0
			END AS math_score
		FROM
			table1
	)
	tmp
GROUP BY
	student_id
```

当然代码也可以更加精炼：

```
SELECT
	student_id,
	MAX(
		CASE
			WHEN course = '语文'
			THEN score
			ELSE 0
		END) AS chinese_score,
	MAX(
		CASE
			WHEN course = '数学'
			THEN score
			ELSE 0
		END) AS math_score
FROM
	table1
GROUP BY
	student_id
```

得到的结果为：

| student_id | chinese_score | math_score |
| ---------- | ------------- | ---------- |
| 1          | 80            | 75         |
| 2          | 95            | 98         |
| 3          | 79            | 90         |



参考资料：

https://analyticshut.com/pivot-rows-to-columns-hive/

https://stackoverflow.com/questions/49112084/transforming-rows-to-columns-in-hive

https://zhuanlan.zhihu.com/p/135440384