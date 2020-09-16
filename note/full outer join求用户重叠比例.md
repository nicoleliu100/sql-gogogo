[toc]

### 背景

full outer join = full join 

之前没有怎么用过这个join，因此对其用法理解的不够深刻。



### 场景

计算三个业务场景的用户重叠比例

![image-20200916112502479](F:\nicole_workspace\nicole_github\sql-gogogo\pics\image-20200916112502479.png)





要求ABC三个场景用户重叠的比例，那么就是判断每个用户是否为ABC场景的用户，然后groupby7种（本来是2^3八种，但是由于排除了非活跃用户，因此000的用户数为0）情况，count 用户id，即目标是得到下面一张表：

| is_场景A_user | is_场景B_user | is_场景C_user | user_cnt |
| ------------- | ------------- | ------------- | -------- |
| 0             | 0             | 1             | 3884     |
| 0             | 1             | 0             | 3545     |
| 0             | 1             | 1             | 0        |
| 1             | 0             | 0             | 107980   |
| 1             | 0             | 1             | 2162     |
| 1             | 1             | 0             | 2483     |
| 1             | 1             | 1             | 209      |

 之前的思路是ABC各个场景的用户union all求出全体用户id，然后分别 left join A 、left join B、left join C。这样就可以通过判断left join后的结果是否为空从而得到下面的table：

| user_id | is_场景A_user | is_场景B_user | is_场景C_user |
| ------- | ------------- | ------------- | ------------- |
| 1111    | 0             | 0             | 1             |
| 2222    | 0             | 1             | 0             |
| 3333    | 0             | 1             | 1             |



但是感觉这样做有点走弯路了，有没有更简单的方法呢？我想到了full outer join，于是趁这个机会复习了一下该方法。



### 用法

和left join 不同，left join 是以左表的key为基准，而full outer join 是两张表的key（key就是 join 的时候 on 条件中的key）都是基准，如果某个key在表A中没有但是在表B中存在，最后该key也是会出现在结果中的。

最后只需要通过coalesce函数从ABC三列中构造出userid即可：

| user_id | a.key | b.key |
| ------- | ----- | ----- |
| 111     | 111   | 111   |
| 222     | 222   | null  |
| 333     | null  | 333   |

```
SELECT  coalesce(场景A.user_id,场景B.user_id,场景C.user_id) AS user_id
        ,CASE    WHEN 场景A.user_id IS NOT NULL THEN 1 
                 ELSE 0 
         END AS is_场景A_user
        ,CASE    WHEN 场景B.user_id IS NOT NULL THEN 1 
                 ELSE 0 
         END AS is_场景B_user
        ,CASE    WHEN 场景C.user_id IS NOT NULL THEN 1 
                 ELSE 0 
         END AS is_star_场景C_user
```





### 参考资料

https://www.dofactory.com/sql/full-outer-join

