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
`hive.exec.reducers.bytes.per.reducer` （每个reduce任务处理的数据量，默认为1000^3=1G）
`hive.exec.reducers.max` （每个任务最大的reduce数，默认为999）

没有强制指定reduce个数,计算reducer数的公式很简单N=min( hive.exec.reducers.max ，总输入数据量/ hive.exec.reducers.bytes.per.reducer )

在hql最后加上distribute by rand()，可强制使hql有reduce过程。

# 数据倾斜
定义：某一个或几个key的数据相比于其他key特别多，导致他们对应的reduce非常慢，其他数据量少的reduce早就执行完了，但是还要等待。
最容易的原因：（1）大量的key为空join连接的情况，空的key都hash到一个reduce上去了.

#
动态分区容易产生多个小文件。

# 参考
https://www.cnblogs.com/xd502djj/p/3799432.html