sql优化

- 可以先使用limit取小样本数据看一下效果
- 使用select fields而不是select *
- 避免使用select distinct

```
SELECT DISTINCT FirstName, LastName, State
FROM Customers
```

```
SELECT FirstName, LastName, Address, City, State, Zip
FROM Customers
```



- 使用 join进行关联而不是 where关联（使用where关联的效果等同于cross join，会创建笛卡尔积Cartesian Product ）

- 使用where进行分组前的过滤，而不是全部放在having中过滤

- 

  

参考资料

https://www.sisense.com/whitepapers/sql-analytics-best-practices-tips-and-tricks/





相关书籍（暂未阅读）：

《SQL优化核心思想》

[https://github.com/liuyu1988/book/blob/master/SQL%E4%BC%98%E5%8C%96%E6%A0%B8%E5%BF%83%E6%80%9D%E6%83%B3.pdf](https://github.com/liuyu1988/book/blob/master/SQL优化核心思想.pdf)