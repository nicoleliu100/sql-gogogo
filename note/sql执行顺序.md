**语句间执行顺序**

SQL Select语句完整的*执行顺序*【从DBMS使用者角度】： 　

1. from子句组装来自不同数据源的数据； 　
2. where子句基于指定的条件对记录行进行筛选； 　
3. group by子句将数据划分为多个分组； 　
4. 使用聚集函数进行计算； 　
5. 使用having子句筛选分组； 　
6. 计算所有的表达式； 　
7. 使用order by对结果集进行排序。 （因为order by 在select后面执行，所以orderby的字段一定要出现在select中，不然就会报错啦）



参考资料

https://www.periscopedata.com/blog/sql-query-order-of-operations

https://www.cnblogs.com/liluxiang/p/10557451.html

