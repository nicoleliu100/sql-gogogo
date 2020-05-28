# 【问题记录】20200528



##问题描述

假如一个表只有一个字段，每行是一个sku_id,现在想把整个表的sku聚合成一个array，要怎么弄呢？



##解题历程

一开始我觉得要用group by ， 然后collect_list。但是只有一列，根本没有用于group by 的字段啊。

我尝试了groupby 1 。无效。

话说我咋会想到用group by1的？？？肯定是因为之前在哪里看到过，那么group by1到底是什么含义呢？我去搜了一下~



排第一的答案是这样的：

> It means to group by the first column regardless of what it's called. You can do the same with `ORDER BY`.

（答案来自:https://stackoverflow.com/questions/7392730/what-does-sql-clause-group-by-1-mean）

哇！原来group by 1是这个意思！也就是指**按第一列分组**。



ok，回归正题，这个问题要怎么解决呢？其实答案很简单，如果最后的期望输出仅仅是一个array的话，直接单独用collect_list()就好啦！！！

如下面代码所示：

```
SELECT    
	CONCAT_WS("#", collect_list(column_a)) 
FROM   
	table_a
WHERE  
	condition1
```



##反思总结

其实这个用法在其他地方也用过的，比如想要count某张表一共有几行，也没有group by，而是直接使用了聚合函数count(*)，如下所示：

```
SELECT    
	count(1) as cnt
FROM   
	table_a
```



所以说，聚合函数的对象可以是整张表，也可以是某个特定的分组：

- 按照表聚合：直接把聚合函数放在select中使用，不需要提前groupby（分组）
- 按照分组聚合：聚合函数和group by搭配使用，按照每个分组来聚合



