有一张记录用户访问时间的表，字段为 pin，date，假设没有重复，即用户即使在某一天多次登录也只有一条记录。

1. 求第一次访问日期是“2020-04-01”的所有用户

    第一种方法：使用**min()函数**直接取第一次访问日期

   ```
   SELECT
   	pin
   FROM
   	(
   		SELECT 
       		pin, 
       		MIN(visit_date) AS first_visit_date 
   		FROM 
   		    table_test 
   		GROUP BY 
   		    pin
   	)
   	tmp
   WHERE
   	first_visit_date = '2020-04-01'
   ```

   第二种方法：使用**窗口函数row_number()over()** 或者 **rank()over()**

   ```
   SELECT
   	pin
   FROM
   	(   -- 先取出所有用户第一次访问日期
   		SELECT
   			pin,
   			visit_date
   		FROM
   			(
   				SELECT
   					pin,
   					visit_date,
   					row_number() over(partition BY pin order by visit_date DESC) AS visit_rank
   				FROM
   					table_test
   			)
   			tmp1
   		WHERE
   			visit_rank = 1
   	)
   	tmp2
   WHERE
   	visit_date = '2020-04-01' --限制第一次访问日期
   ```

   

2. 求在“2020-04-01”访问的新用户第二天的留存率。

    所谓留存率，是指在T日访问的新用户，在T+1日也访问了的比例。

    思路是构建一张**类似**于这样的表：

    | pin  | is_fisrt_visit_0401 | is_visited_0402 |
    | ---- | ------------------- | --------------- |
    | A    | 1                   | 1               |
    | B    | 1                   | null            |

    SQL: 最终取的不1和null，但是不影响结果

    ```
    SELECT
    	COUNT(a.pin) / COUNT(is_visit_next_day) AS rentention_rate
    FROM
    	(
    		SELECT
    			pin,
    			first_visit_date
    		FROM
    			(
    				SELECT 
        				pin, 
        				MIN(visit_date) AS first_visit_date 
    				FROM 
    				    table_test 
    				GROUP BY 
    				    pin
    			)
    			tmp
    		WHERE
    			first_visit_date = '2020-04-01'
    	)
    	a
    LEFT JOIN
    -- 第二张表是第二天访问的用户
    	(
    		SELECT 
    		    pin, 
    		    visit_date as is_visit_next_day 
    		FROM 
    		    table_test 
    		WHERE 
    		    visit_date = '2020-04-02'
    	)
    	b a.pin = b.pin
    ```

3. 求从“2020-04-01”开始的未来30天内，每一天新用户的次日留存率。

思路是构建一张类似于这样的表：

| pin  | first_visit | is_visit_next_day |
| ---- | ----------- | ----------------- |
| A    | 04-01       | yes               |
| B    | 04-02       | no                |

左边两个字段好得到：只需要限制第一次访问时间为0401-0430期间内即可，第三个字段也可以通过限制在这个时间范围内访问过条件得到。然后把左表和右表通过key pin join起来即可。注意是left join~

代码如下：

```
SELECT
	first_visit_date,
	COUNT(a.pin) / COUNT(is_visit_next_day) AS rentation_rate
FROM
	(
		SELECT
			a.pin,
			a.first_visit_date,
			b.is_visit_next_day
		FROM
			(
				SELECT
					pin,
					first_visit_date
				FROM
					(
						SELECT pin, MIN(visit_date) AS first_visit_date FROM table_test GROUP BY pin
					)
					tmp
				WHERE
					first_visit_date >= '2020-04-01'
					AND first_visit_date <= '2020-04-30'
			)
			a
		LEFT JOIN
			-- 第二张表是第二天访问的用户
			(
				SELECT
					pin,
					visit_date AS is_visit_next_day
				FROM
					table_test
				WHERE
					visit_date >= '2020-04-02'
					AND visit_date <= '2020-05-01'
			)
			b a.pin = b.pin
	)
GROUP BY
	first_visit_date
```

