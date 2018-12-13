---
title: hive笔记2-深入
tags:
  - hadoop
  - hive
categories:
  - hadoop
copyright: true
date: 2018-09-19
top: 1
---
# reduce数量
一个reduce生成一个文件。
可以在hive运行sql的时，打印出来，如下：
```
Number of reduce tasks not specified. Estimated from input data size: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapred.reduce.tasks=<number>
```
reduce数量由以下三个参数决定，
`mapred.reduce.tasks` (强制指定reduce的任务数量)
`hive.exec.reducers.bytes.per.reducer` （每个reduce任务处理的数据量，默认为1000^3=1G,一般可能会不管用。）
`hive.exec.reducers.max` （每个任务最大的reduce数，默认1009）

没有强制指定reduce个数,计算reducer数的公式很简单N=min(`hive.exec.reducers.max`, 总输入数据量/`hive.exec.reducers.bytes.per.reducer`)
在hql最后加上`distribute by rand()`，可强制使hql有reduce过程。

https://www.iteblog.com/archives/1697.html

### map个数计算
gzip不支持切片，因此一个gzip压缩文件不能通过切片 由多个map执行。
`minSize=max{minSplitSize, mapred.min.split.size}`（minSplitSize大小默认为1B）
`maxSize=mapred.max.split.size`（不在配置文件中指定时大小为`Long.MAX_VALUE=3G`）
`splitSize=max{minSize, min{maxSize, blockSize}}`
一般来说，一个map不能处理多个文件。
在一个map不涉及到多文件处理时，用上面的参数。想增加map个数时，把`maxSize`调小，小于`blockSize`（不同版本默认大小不同，2.X一般是128M）时可以取代`blocksize`变成新的切片大小`splitsize`。

**_map数是这样计算方式:_**
文件大小/splitSize>1.1，创建一个split0，文件剩余大小=文件大小-splitSize
　.....
剩余文件大小/splitSize<=1.1 将剩余的部分作为一个split
每一个分片对应一个map任务，这样map任务的数目也就显而易见啦。 

但其实**_一个map可以跨文件处理_**：
通过实验，gzip、orc、lzo都支持合并。
一般在想减少map个数，但是文件大小都小于`blockSize`时，上面已经使不上劲时使用下面参数。
`set hive.hadoop.supports.splittable.combineinputformat=true;` 开关
`set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;` 执行Map前进行小文件合并
`set mapred.max.split.size=2048000000;` 2G 每个Map最大输入大小  
`set mapred.min.split.size.per.node=2048000000;` 一个节点上split的至少的大小 ，决定了多个data node上的文件是否需要合并
`set mapred.min.split.size.per.rack=2048000000;` 一个交换机下split的至少的大小，决定了多个交换机上的文件是否需要合并
详细看：[小文件合并](/2018/10/19/hive小文件合并/)

# 数据倾斜
定义：某一个或几个key的数据相比于其他key特别多，导致他们对应的reduce非常慢，其他数据量少的reduce早就执行完了，但是还要等待。
最容易的原因：（1）大量的key为空join连接的情况，空的key都hash到一个reduce上去了.

#
动态分区容易产生多个小文件。

# 参考
https://www.cnblogs.com/xd502djj/p/3799432.html
https://www.cnblogs.com/yueliming/p/3251285.html