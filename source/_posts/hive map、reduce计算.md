---
title: hive map、reduce计算
tags:
  - hadoop
  - hive
categories:
  - hadoop
copyright: true
date: 2018-09-19
---
### map个数计算
splitsize= Math.max(minSize, Math.min(goalSize, blockSize)),通常这个值=blockSize，输入的文件较小，文件字节数小于blocksize时，splitsize=输入文件字节数之和。

**_gzip不支持切片，因此一个gzip压缩文件不能通过切片 由多个map执行，只能是有多少个文件，对应有多少个map。_**
`minSize=max{minSplitSize, mapred.min.split.size}`（minSplitSize大小默认为1B）
`maxSize=mapred.max.split.size`（不在配置文件中指定时大小为`Long.MAX_VALUE=3G`）
`splitSize=max{minSize, min{maxSize, blockSize}}`
一般来说，一个map不能处理多个文件。
在一个map不涉及到多文件处理时，用上面的参数。想增加map个数时，把`maxSize`调小，小于`blockSize`（不同版本默认大小不同，2.X一般是128M）时可以取代`blocksize`变成新的切片大小`splitsize`。

**_map计算方式:_**
文件大小/splitSize>1.1，创建一个split0，文件剩余大小=文件大小-splitSize
　.....
剩余文件大小/splitSize<=1.1 将剩余的部分作为一个split
每一个分片对应一个map任务，这样map任务的数目也就显而易见啦。 

但其实**_一个map可以跨文件处理_**：
通过实验，gzip、orc、lzo都支持文件合并。
一般在想减少map个数，但是文件大小都小于`blockSize`时，上面已经使不上劲时使用下面参数。
`set hive.hadoop.supports.splittable.combineinputformat=true;` 开关
`set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;` 执行Map前进行小文件合并
`set mapred.max.split.size=2048000000;` 2G 每个Map最大输入大小  
`set mapred.min.split.size.per.node=2048000000;` 一个节点上split的至少的大小 ，决定了多个data node上的文件是否需要合并
`set mapred.min.split.size.per.rack=2048000000;` 一个交换机下split的至少的大小，决定了多个交换机上的文件是否需要合并
**_此时，map计算方式:_**
a、根据输入目录下的每个文件,如果其长度超过mapred.max.split.size,以block为单位分成多个split(一个split是一个map的输入),每个split的长度都大于mapred.max.split.size, 因为以block为单位, 因此也会大于blockSize, 此文件剩下的长度如果大于mapred.min.split.size.per.node, 则生成一个split, 否则先暂时保留.
b、现在剩下的都是一些长度效短的碎片,把每个rack下碎片合并, 只要长度超过mapred.max.split.size就合并成一个split, 最后如果剩下的碎片比mapred.min.split.size.per.rack大, 就合并成一个split, 否则暂时保留.
c、把不同rack下的碎片合并, 只要长度超过mapred.max.split.size就合并成一个split, 剩下的碎片无论长度, 合并成一个split.
举例: mapred.max.split.size=1000
mapred.min.split.size.per.node=300
mapred.min.split.size.per.rack=100
输入目录下五个文件,rack1下三个文件,长度为2050,1499,10, rack2下两个文件,长度为1010,80. 另外blockSize为500.
经过第一步, 生成五个split: 1000,1000,1000,499,1000. 剩下的碎片为rack1下:50,10; rack2下10:80
由于两个rack下的碎片和都不超过100, 所以经过第二步, split和碎片都没有变化.
第三步,合并四个碎片成一个split, 长度为150.
如果要减少map数量, 可以调大mapred.max.split.size, 否则调小即可.
其特点是: 一个块至多作为一个map的输入，一个文件可能有多个块，一个文件可能因为块多分给做为不同map的输入， 一个map可能处理多个块，可能处理多个文件。
详细看：[小文件合并](/2018/10/19/hive小文件合并/)

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

 只有一个reduce的场景：
  a、没有group by 的汇总，比如把select pt,count(1) from popt_tbaccountcopy_mes where pt = '2012-07-04' group by pt; 写成 select count(1) from popt_tbaccountcopy_mes where pt = '2012-07-04';
  b、order by
  c、笛卡尔积
  
https://www.iteblog.com/archives/1697.html

# shuffle
讲的很棒：https://www.iteblog.com/archives/1119.html
https://www.cnblogs.com/ljy2013/articles/4435657.html


# 数据倾斜
定义：某一个或几个key的数据相比于其他key特别多，导致他们对应的reduce非常慢，其他数据量少的reduce早就执行完了，但是还要等待。
最容易的原因：（1）大量的key为空join连接的情况，空的key都hash到一个reduce上去了.

#
动态分区容易产生多个小文件。

# 参考
https://www.cnblogs.com/xd502djj/p/3799432.html
https://www.cnblogs.com/yueliming/p/3251285.html
