## 背景介绍

之前建的表都是一个分区，每次insert也只是insert一个分区的数据，因此都只是习惯性地使用insert overwrite。这次需要把用样的一份数据insert到两个不同分区，且这同一份数据来自于两个独立的任务，这才开始斟酌insert into和insert overwrite的区别。



##问题拆解

**问题1**

首先把上述问题简化一下，一张表的某个分区的数据来自于两个独立执行的任务，该用insert into还是insert overwrite？

**问题2**

然后是在某张表（table A）某个分区数据（partition1）已经产生了的情况下，想把这份数据重新insert到这张表的另一个分区（partition2）中。这时该用insert into还是insert overwrite？



### 自问自答

要回答上面两个问题，首先必须清楚insert into和insert overwrite的区别。

**insert的对象有两种**

首先我们要清楚，insert的**对象**是一张**表**或者**表的某个分区**：

- 如果表不是分区表：由于没有分区，因此insert是针对这张表而言的，修改的主体这张表。
- 如果表是分区表：对于分区表，insert时一定要指定要插入的分区是什么，也就是说这时insert的对象是这张表的指定分区，修改的是这个分区的数据，对于其他分区的数据没有影响。

**into和overwrite的区别**

- insert into：往表或者某个分区中**追加数据**，原有数据不受影响
  - insert into = 直接追加数据（append）
- insert overwrite：往表或者某个分区中**重写数据**。重写意味着表中原本的所有数据都会被新数据覆盖掉。
  - insert overwrite = 删除原本的数据 + 加入新数据 （delete+append）

**if not exisits**

这个关键词相当于加了一道保险。对于十分重要的数据，如果不想不小心错误操作，可以加上这个关键词。

`insert overwrite  table tablename （partition ） if not exists`  

这样的话，如果原本没有这个分区才往这个分区写数据，如果原本存在这个分区就不会做任何修改啦。



**针对上面两个问题的回答**

基础知识了解后，再来看上面两个问题。

问题1：

这两个任务的数据合起来才是一个分区的完整数据，所以显然不能用insert overwtite，只能用insert into才能保证分区的数据不被覆盖。

问题2：

1. 把一份数据重新写到另一个分区中，如果任务是一次性的，且这个新分区之间不存在于这个表中，那么insert into和insert overwrite都是可以的。
2. 如果任务固化成每天执行的，那么这个新分区已经存在于原表中了，我们需要覆盖原有数据，将新数据插入其中，这时要使用insert overwrite。
3. 综上，使用insert overwrite。



###  参考资料

https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML