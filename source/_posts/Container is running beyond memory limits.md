---
title: 'Container is running beyond memory limits'
date: "2018/06/20"
tags: [hive内存溢出]
categories: [hadoop]
copyright: true
---
报错：
`is running beyond physical memory limits. Current usage: 3.1 GB of 3 GB physical memory used; 4.9 GB of 60 GB virtual memory used. Killing container.`

解决：
`set mapreduce.map.memory.mb=4096` 或者8196.看hive的输出只有map，因为只设置它的大小。

`set mapreduce.reduce.memory.mb=8192` ； 设置reduce大小。

参考：https://stackoverflow.com/questions/21005643/container-is-running-beyond-memory-limits


报错：
`The maximum number of dynamic partitions is controlled by hive.exec.max.dynamic.partitions and hive.exec.max.dynamic.partitions.pernode. Maximum was set to: 100`
原因看报错，显而易见，动态分区超过最大动态分区，默认100.
解决：
```mysql
SET hive.exec.max.dynamic.partitions=2048;
SET hive.exec.max.dynamic.partitions.pernode=256;
```
参考：https://blog.csdn.net/oDaiLiDong/article/details/49884571