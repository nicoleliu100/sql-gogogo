- count(1) 会对全表进行扫描，统计所有记录的条数；（**和count()没有区别**）*
- *count(*) 会对全表进行扫描，统计所有记录的条数；
- count(列名)统计该字段**不为null**的记录条数
  - 即 count(<column>) needs to compare the column value to NULL，所以会稍微慢一点



参考资料：https://stackoverflow.com/questions/1221559/count-vs-count1-sql-server