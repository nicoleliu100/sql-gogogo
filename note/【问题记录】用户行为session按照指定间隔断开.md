### 背景

在item2vec中，首先要构建用户行为序列（比如用户连续浏览sku的序列、用户下载app的序列等）。下面以用户浏览sku为例，分析一下问题的背景。

由于用户行为可能是在短期内迅速变化着的。有可能用户这一次访问是想买牛奶，下一次访问（10min后）则变成了想买手机。

我们构建的有效序列是基于：**用户在这一次序列内浏览的所有SKU都是相关联的**这个假设。因此一般会根据业务情况为这些序列加上断开的条件从而来保证我们的数据是干净的（去噪），如间隔时间超过30min，则认为是两次不同的序列。



### 思路

（20200608）

那么具体sql怎么写呢？

其实我之前写过一版，当时是在关联分析（fpgrowth）中，实现的也是类似的需求，只是我当时没有直接设置固定的分割间隔，而是直接**认为（假设）用户在一次访问中（每一次唤醒app）的行为是相关联的**。

所以那时的方法是：

- 首先根据用户埋点表求得用户每一次访问app时间戳，
- 并用**lead()over()**窗口函数构造出每次访问的时间起始点和结束点，
- 然后把每一次用户行为对应到相应的访问次序中，
- 最后按照访问次序分组，得到用户浏览sku的列表。



今天又接触到类型的需求，但是想尝试一下直接设定分隔时间的方法，居然没有马上想出解决办法。后来想了一会儿，觉得我其实可以换到spark中，这样可以使用udf，所以下面考虑的是：

- 如果直接用sql怎么写？
- 如果我使用pyspark，可以借助udf，要怎么写？



### 代码



#### 直接用sql

我自己没想出来。。。。咨询了sql大神同事后，对他的思路佩服不已。。。

- 先用lag()over()移位 注意这里用lag而不是lead，用lead会有点问题
- 然后time 和 lag后的time求差
- **重点来了！！！这一步我愣是没想到，用sum()over()来构建分组的key**



举个例子，假设设置的interval = 5s：

| 时间点（s） | lag移位时间点（s） | time_diff | is time diff greater than interval(5s)? | sum()over()左边那一列 |
| ----------- | ------------------ | --------- | --------------------------------------- | --------------------- |
| 1           | null               | 1         | 0                                       | 0                     |
| 2           | 1                  | 1         | 0                                       | 0                     |
| 4           | 2                  | 2         | 0                                       | 0                     |
| 100         | 4                  | 96        | 1                                       | 1                     |
| 101         | 100                | 1         | 0                                       | 1                     |
| 150         | 101                | 49        | 1                                       | 2                     |



然后**用最后一列作为group by的key**，就可以将原始数据分成三个数组了！！！！





#### 使用pyspark

- 把用户的浏览拼成一个list，格式为 [time1#id1, time2#id2,...]

- 然后定义一个split_session函数对这个list进行处理
  - 遍历数组，求当前位置的time和下一个位置的time之差
  - 根据时间差和自定义间隔之间的大小关系作出相应的处理逻辑

```
def split_session(collect_coupon_list):
	”“”
	collect_coupon_list like  [time1#id1, time2#id2,...]
	“”“
    # 超过5min 则断开
    interval = 5
    
    res = []
    len_list = len(collect_coupon_list)
    
    # 用continous_session保存相邻间隔始终小于5min的记录
    continous_session = []
    
    # 开始遍历
    for i in range(len_list-1):
        cur_collect_time = datetime.datetime.strptime(collect_coupon_list[i].split("#")[0], "%Y-%m-%d %H:%M:%S.%f")
        next_collect_time = datetime.datetime.strptime(collect_coupon_list[i+1].split("#")[0], "%Y-%m-%d %H:%M:%S.%f")
        # 两次间隔的time_diff
        time_diff = next_collect_time - cur_collect_time
        
        if  time_diff <= datetime.timedelta(seconds=interval*60):
            continous_session.append(collect_coupon_list[i].split("#")[1])
        else:
            res.append(continous_session)
            continous_session = []
    
    continous_session.append(collect_coupon_list[len_list-1].split("#")[1])
    res.append(continous_session)        
    return res
```

自测OK~

