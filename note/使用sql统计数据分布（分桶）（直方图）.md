







```
select
    bucket_floor,
    CONCAT(bucket_floor, ' to ', bucket_ceiling) as bucket_name,
    count(*) as count
from (
	select 
		floor(revenue/5.00)*5 as bucket_floor,
		floor(revenue/5.00)*5 + 5 as bucket_ceiling
	from web_sessions_table
) a
group by 1, 2
order by 1;
```





```
SELECT 
  bucket_floor, 
  CONCAT(
    bucket_floor, ' to ', bucket_ceiling
  ) AS bucket_name, 
  COUNT(user_id) AS user_count 
FROM 
  (
    SELECT 
      user_id, 
      floor(order_monetary / 5.00)* 5 AS bucket_floor, 
      floor(order_monetary / 5.00)* 5 + 5 AS bucket_ceiling 
    FROM 
      (
        SELECT 
          USER.user_id AS user_id, 
          CASE WHEN order_monetary is not null THEN order_monetary ELSE 0 END AS order_monetary 
        FROM 
          (
            SELECT 
              user_id 
            FROM 
              dwd_app_user_info 
            WHERE 
              STATUS = 1
          ) USER 
          LEFT JOIN (
            SELECT 
              from_user_id, 
              sum(total_amount) AS order_monetary 
            FROM 
              dwd_app_trade_flow_currency 
            WHERE 
              dt >= '20200817' 
              AND dt <= '20200823' 
            GROUP BY 
              from_user_id
          ) rfm ON USER.user_id = rfm.from_user_id
      ) tmp1
  ) tmp2 
GROUP BY 
  bucket_floor, 
  bucket_ceiling;
```



https://web.archive.org/web/20180205094300/https://www.wagonhq.com/sql-tutorial/creating-a-histogram-sql